# Command Line

All about command line on Linux.

References:

- [Art of Command Line](https://github.com/jlevy/the-art-of-command-line)
- [How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676)
- Personal snippets

## Softlinks & Hardlinks

Softlinks are just a reference to a file path existing somewhere. Editing it, will not affect the file it's linked to.

Hardlinks in turn is a direct reference to the target file path, thus, editing it will also edit the file it's linked to.

```bash
ln -s file-path.txt ~/soft-symlinked-file.txt
```

If target file is deleted, soft symlinked file will still exist but will be broken.

```bash
ln file-path.txt ~/hard-symlinked-file.txt
```

If target file is deleted, hard links will still be there with the latest content.

## Open terminal commands in EDITOR

ctrl-x ctrl-e will open the current command in an EDITOR for multi-line editing

## NOHUP (allow process to stay alive on terminal closing)

```bash
nohup command &
```

See also, `jobs`, `disown`, and `command &`

## Run on command while staying on same DIR

```bash
# do something in current dir
(cd /some/other/dir && other-command)
# continue in original dir
```

## Output of a command, can be treated as a file, within `<()`

```bash
diff ~/some_file.txt <(ssh host && cat ~/some_file.txt)
```

## SSH Config

Few optimizations on SSH config. > settings contains config to avoid dropped connections in certain network environments, uses compression (which is helpful with scp over low-bandwidth connections), and multiplex channels to the same server with a local control file:

```bash
  TCPKeepAlive=yes
  ServerAliveInterval=15
  ServerAliveCountMax=6
  Compression=yes
  ControlMaster auto
  ControlPath /tmp/%r@%h:%p
  ControlPersist yes
```

## 128KB limit on command line

This "Argument list too long" error is common when wildcard matching large numbers of files. (When this happens alternatives like find and xargs may help.)

## Tools for working with json, yaml, csv

[JSON](https://stedolan.github.io/jq/)
[YAML](https://github.com/0k/shyaml)
[CSV](https://csvkit.readthedocs.io/en/latest/)

## ZCAT

```bash
Print data from gzip compressed files.

- Print the uncompressed contents of a gzipped file to the standard output:
  zcat file.txt.gz

- Print compression details of a gzipped file to the standard output:
  zcat -l file.txt.gz
```

See also `zless`, `zmore`, `zgrep`.

## Best practices on shell scripts

[Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/)

## grep alternatives

`ack` `ag` `ripgrep`

## xargs

takes whitespace separated strings from stdin and converts into command-line arguments

`echo "home tmp" | xargs ls` 

will run `ls home tmp`

### useful tips

`xargs -n 1` will run a separate process per each input. Pass in `-P n` to set max number of parallel processes

## awk

tiny language to manipulate columns of data.

Example:

```bash
ps | awk -F, '{print $3}
```

## sed

most often used for replacements

```bash
sed s/match/replace/g file.txt
```

```bash
sed 5d # deletes the 5th line
```

```bash
sed /cat/d # deletes the line matching cat
```

```bash
sed -n 5,30p # prints lines between 5 to 30
```

```bash
sed 'i 17 panda' # insert panda on line 17
```

```bash
sed G # double space a file
```

## bash tricks

- Commands that start with `space` does not go to the history

- !! run the last command ran as sudo

- ctrl + r : search history

- ctrl + z : suspend a process, `fg` to get back to it in foreground `bg` in background

- `fc` : fix command, opens the last ran command in $EDITOR

## disk usage

```bash
du # tells how much diskspace a /directory occupies
  -h # human readable format
  -s # summary

df # tells the free space over each partitions
  -h # human readable format

ncdu # interactive prompt going over files/directories showing the sizes
  -h # human readable format
```

## watch

rerun commands intermittently (2 seconds by def)

## cal

calendar

## xsel/xclip

paste contents into terminal from system clipboard

## comm

find common lines in two sorted files

## file

figure out the mime details of the file

## ts

add timestamps

## kill

kill is not only used to kill processes. It can be used to send any signal to any program

### kill -l

list all signals

### pkill

kills a process based on pattern matched and pid found

`pkill -f firefox`

### killall -SIGNAL name

signals all processes to be killed with NAME

useful flags:

- -w : wait for process to die
- -i : ask before signalling

## zcat

cats a gzipped file

## lsof

lists open files

### lsof -i

lists open network sockets

### lsof | grep deleted

lists deleted files

### netstat

another way of showing open sockets

