# Ruby I/O

This post covers most of the practical stuff I need regarding UNIX I/O and Ruby's relationship with it. Here is some pre | post reading material to keep refreshed often.

- [Overview of FREEBSD I/O](https://www.freebsd.org/doc/en/books/design-44bsd/overview-io-system.html)
- [TTY System](https://www.linusakesson.net/programming/tty/)
- [Thoughtbot's post on Ruby I/O](https://thoughtbot.com/blog/io-in-ruby)
- [Repo to practice Ruby I/O](https://github.com/elm-city-craftworks/course-001)
- [Ruby I/O documentation](https://ruby-doc.org/core-3.0.0/IO.html)

## Standard I/O streams and the filesystem

### Making ruby scripts runnable as command line utilities

1. Add sheebang

    ```ruby
    #!/usr/bin/env ruby

    ...your script
    ```

2. Make it executable

    `chmod +x bin/yourscript`

3. Add to PATH

    export PATH=/path/to/script/folder:$PATH

### ARGV object: for collecting parameters

Ruby stores the parameters passed into command line programs with `ARGV` array.

```ruby
#!/usr/bin/env ruby

puts ARGV.inspect
```

```shell
./filename -s first second third

["-s", "first", "second", "third"]
```

### ARGF object: for joining streams

Detailed write-up [here](https://thoughtbot.com/blog/rubys-argf)

Ruby provides standard `ARGF` object which combines different streams of input into a single one -- or fallsback to STDIN in case no file was given.

For example, these two scripts would do the same.

```ruby
# Script 1
if ARGV.length > 0
  ARGV.each do |filename|
    puts File.read(filename)
  end
else
  puts STDIN.read
end

# Script 2
ARGF.read
```

> Be mindful when your program accepts options such as --color, --size etc... `ARGV` items will be treated as filenames, thus `ARGF` will try to read them as such -- resulting in file not found. It's good idea to remove them before using ARGF.

Command line tool `cat` supports both passing filenames and standard input stream at the same time. This can be achieved by special filename `-`.

```shell
echo "Standard Input Stream" > cat file1.txt - file2.txt

File 1 Content
Standard Input Stream
File 2 content
```

`ARGF` also supports this ^.

`ARGF` also provides helpful methods such as -- looking up for the filename, or I/O stream it's operating on.

### OptionParser class - [API Docs](https://ruby-doc.org/stdlib-1.9.2/libdoc/optparse/rdoc/OptionParser.html#method-i-parse)

Parse options for your command line tool. Add --help, callbacks on the passed options and more...

```ruby
require "optparse"

puts "ARGV is #{ARGV.inspect}"

params = {}
parser = OptionParser.new 

parser.on("-n") { params[:line_numbering_style] ||= :all_lines         }
parser.on("-b") { params[:line_numbering_style]   = :significant_lines }
parser.on("-s") { params[:squeeze_extra_newlines] = true               }

files = parser.parse(ARGV)

puts "params are #{params.inspect}"
puts "files are #{files.inspect}"
```

### Error handling and exit codes

In case of errors, always write to STDERR and exit the program with exit code 1. This will make sure your program will follow the UNIX philosophy and play nice with other applications or tools.

The `Open3.capture3` method runs a shell command and then returns whatever was written to STDOUT and STDERR, as well as a Process::Status, similar to `$?`.

```ruby
require 'open3'

command_stdout, command_stderr, command_status = Open3.capture3('command --someoption')
```

### Assignment: `ls` clone in ruby

Assignment Description [here](https://github.com/elm-city-craftworks/course-001/blob/master/PART_1.md)

My implementation source code [here](https://github.com/gadimbaylisahil/ruby-ls-clone/blob/main/ruby-ls)

## Socket programming and network I/O

Basic HTTP Server

```ruby
require 'socket' # Provides TCPServer and TCPSocket classes

# Initialize a TCPServer object that will listen
# on localhost:2345 for incoming connections.
server = TCPServer.new('localhost', 2345)

# loop infinitely, processing one incoming
# connection at a time.
loop do

  # Wait until a client connects, then return a TCPSocket
  # that can be used in a similar fashion to other Ruby
  # I/O objects. (In fact, TCPSocket is a subclass of IO.)
  socket = server.accept

  # Read the first line of the request (the Request-Line)
  request = socket.gets

  # Log the request to the console for debugging
  STDERR.puts request

  response = "Hello World!\n"

  # We need to include the Content-Type and Content-Length headers
  # to let the client know the size and type of data
  # contained in the response. Note that HTTP is whitespace
  # sensitive, and expects each header line to end with CRLF (i.e. "\r\n")
  socket.print "HTTP/1.1 200 OK\r\n" +
               "Content-Type: text/plain\r\n" +
               "Content-Length: #{response.bytesize}\r\n" +
               "Connection: close\r\n"

  # Print a blank line to separate the header from the response body,
  # as required by the protocol.
  socket.print "\r\n"

  # Print the actual response body, which is just "Hello World!\n"
  socket.print response

  # Close the socket, terminating the connection
  socket.close
end
```

Full source code [here](https://github.com/elm-city-craftworks/practicing-ruby-examples/tree/master/v7/002)

- TCP Socket: Allows basic I/O operations on a network between client and a server. OS provides low level API for sockets which is wrapped by the higher level programming languages.

- Ruby's I/O class provides the most comprehensive stream processing behaviour. Sockets are treaded similarly to files for basic reading, writing and stream processing operations. This is coming from the notion of treating everything like a file in UNIX. Read more [here](https://www.howtogeek.com/117939/htg-explains-what-everything-is-a-file-means-on-linux/)

- There are two distinct types of sockets - Connection and Listening sockets. Clients only need to make use of connection sockets while servers can make use of both. When a client communicates with the TCP server, it creates a socket by specifying IP and the port, then tries to establish a connection. 

- On the server side the listening socket is bound to a particular port, and its job is to process client's incoming requests. Once the request is processed, it creates a connection socket.

- Main difference is that the in listening sockets the `port` is explicitly defined and well known in advance whereas connection sockets are assigned ports based on [`ephemeral port range`](https://en.wikipedia.org/wiki/Ephemeral_port)

- It's important to close the sockets explicitly to free resources and make sure connection does not send or wait for any response. Nevertheless, Ruby GC will close any connections that are not references anymore itself.
