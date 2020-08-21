# Debian Administration Handbook

## dpkg, APT and ar

ar: handles .deb files, `ar t archive` shows the list of files contained in such
    an archive, `ar x archive` extracts the file from the archive into the
    current working directory, `ar d archive file` deletes a file from the
    archive. It's very useful when recovering `dpkg` deteled by mistake.



## Backup

  using `rsync`, prceded by a duplication of the content of the previous backup
with hadr links, which prevents usage of too much hard drive space. And only
process then only replaces files that have been modified since last backup.

hard link, as opposed to a symbolic link, same as giving an existing file a
second name. Deletion of a hard link only removes one of the name associated
with the file, not removes the file itself. And the hard link can only be
created on the same filesystem, while symbolic links are no subject to this
limitation.