# Command Line

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
