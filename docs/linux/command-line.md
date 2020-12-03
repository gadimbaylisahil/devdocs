# Command Line

Some personal & some from [Art of Command Line](https://github.com/jlevy/the-art-of-command-line)

## Difference between soft and hard symlinks

Softlinks are just a reference to a file path existing somewhere. Editing it, will not affect the file it's linked to.

Hardlinks in turn is a direct reference to the target file path, thus, editing it will also edit the path it's linked to.

```bash
ln -s file-path.txt ~/soft-symlinked-file.txt
```

If target file is deleted, soft symlinked file will still exist but will be broken.

```bash
ln file-path.txt ~/hard-symlinked-file.txt
```

If target file is deleted, hard links will still be there with the latest content.

## Open long terminal command in vim

ctrl-x ctrl-e will open the current command in an editor for multi-line editing.

## Allow process to live when terminal is killed(bg process)

```bash
nohup command
```

See also, `jobs`, `disown`, and `command &`

## Run a command while staying on the same dir

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

## Be careful about 128KB limit on command line

This "Argument list too long" error is common when wildcard matching large numbers of files. (When this happens alternatives like find and xargs may help.)

## Tools for working with json, yaml, csv

[JSON](https://stedolan.github.io/jq/)
[YAML](https://github.com/0k/shyaml)
[CSV](https://csvkit.readthedocs.io/en/latest/)

## ZCAT

```bash
zcat

Print data from gzip compressed files.

- Print the uncompressed contents of a gzipped file to the standard output:
  zcat file.txt.gz

- Print compression details of a gzipped file to the standard output:
  zcat -l file.txt.gz
```

See also `zless`, `zmore`, `zgrep`.

## Best practices on shell scripts

[Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/)