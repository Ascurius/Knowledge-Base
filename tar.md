# Using TAR for creating backups

### Basic options

- `-c` create a backup
- `-f` specify the destination name of the backup file
- `-z` states that the `gzip` compression should be used during the backup process
- `-v` use for more detailed verbose information. Can even be used twice as `-vv` to get even more detailed information.
- `-P` run the backup in absolute mode, meaning that all paths are given as absolute paths

### Naming conventions

Note that by convention an A is included in the file name suffix, when using absolute path mode.

When using compression, make sure to append the compression algorithm acronym to the suffix. For example, when using `gzip`, append `gz` to the file name suffix.

## Exmaples

#### Determine the size of a backup in kilobytes

```bash
tar czf - /directory/to/backup/ | wc -c
```

#### Create simple tar backup

```bash
tar cvfP /backups/home.133.A.tar /home/
```
#### Verify contents of a backup archive
```bash
tar -tvf backup.tar.gz
```
This command should list all files and directories of the backup archive to the command line.
#### Create Incremental Backup

##### 1. Create full backup
First you need to create a full backup as a basis for further incremental backups. To to this run the command

```bash
tar -czvg backup.snapshot -f backup.tar.gz /path/to/backup
```
where
- `backup.snapshot` is the name of an index file where `tar` stores the list of files and directories.
- `backup.tar.gz` is the name of the backup archive
- `/path/to/backup` is the directory which you want to backup

##### 2. Create incremental backup

```bash
tar -czvg backup.snapshot -f incremental-backup-01.tar.gz /path/to/backup
```
##### 3. Inspect content of backup
To inspect the directory structure at a given incremental backup, use the following command:

```bash
tar -Gtvv  -f incremental-backup-03.A.tar.gz
```

### References

- https://docs.rockylinux.org/books/admin_guide/09-backups/#backing-up-with-tar
- https://www.computernetworkingnotes.com/linux-tutorials/create-and-restore-incremental-backups-in-linux-with-tar.html