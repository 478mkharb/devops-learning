# tar Command in Linux and File Compression Types

## 1. What is tar in Linux

`tar` stands for **Tape Archive**.

It is used to **combine multiple files and directories into a single archive file**.

Important:

`tar` by itself **does NOT compress files**.

It only **archives** them into a single file.

Compression is done using tools like:

* gzip
* bzip2
* xz

So typically tar is used together with compression utilities.

---

## 2. Basic tar Syntax

```
tar [options] archive_name files
```

Example:

```
tar -cvf archive.tar file1 file2
```

---

## 3. Common tar Options

| Option | Meaning                 |
| ------ | ----------------------- |
| c      | create archive          |
| x      | extract archive         |
| v      | verbose (show progress) |
| f      | file name of archive    |
| t      | list contents           |
| z      | use gzip compression    |
| j      | use bzip2 compression   |
| J      | use xz compression      |

---

## 4. Creating a tar Archive

Example:

```
tar -cvf backup.tar folder/
```

Explanation:

* c → create archive
* v → verbose output
* f → archive file name

Result:

```
backup.tar
```

This file contains all files from the folder.

---

## 5. Extracting tar Archive

```
tar -xvf backup.tar
```

Explanation:

* x → extract
* v → verbose
* f → archive file

---

## 6. Viewing Contents of tar

```
tar -tvf backup.tar
```

This lists all files inside the archive without extracting them.

---

# Types of File Compression in Linux

Linux supports multiple compression algorithms.

The most common ones used with tar are:

* gzip
* bzip2
* xz

---

# 1. tar + gzip Compression

File extension:

```
.tar.gz
```

or

```
.tgz
```

Example:

```
tar -czvf backup.tar.gz folder/
```

Options:

* z → gzip compression

Extract:

```
tar -xzvf backup.tar.gz
```

---

# 2. tar + bzip2 Compression

File extension:

```
.tar.bz2
```

Example:

```
tar -cjvf backup.tar.bz2 folder/
```

Options:

* j → bzip2 compression

Extract:

```
tar -xjvf backup.tar.bz2
```

---

# 3. tar + xz Compression

File extension:

```
.tar.xz
```

Example:

```
tar -cJvf backup.tar.xz folder/
```

Options:

* J → xz compression

Extract:

```
tar -xJvf backup.tar.xz
```

---

# Compression Comparison

| Compression | Speed   | Compression Ratio | Common Extension |
| ----------- | ------- | ----------------- | ---------------- |
| gzip        | Fast    | Medium            | .tar.gz          |
| bzip2       | Slower  | Better            | .tar.bz2         |
| xz          | Slowest | Best compression  | .tar.xz          |

---

# Real DevOps Examples

## Backup Logs

```
tar -czvf logs_backup.tar.gz /var/log
```

---

## Backup Application Folder

```
tar -czvf app_backup.tar.gz /opt/application
```

---

## Extract Backup

```
tar -xzvf app_backup.tar.gz
```

---

# Quick Reference

Create archive

```
tar -cvf file.tar folder
```

Create gzip archive

```
tar -czvf file.tar.gz folder
```

Extract

```
tar -xvf file.tar
```

Extract gzip

```
tar -xzvf file.tar.gz
```

List contents

```
tar -tvf file.tar
```

---

# Interview Explanation

The `tar` command in Linux is used to archive multiple files and directories into a single file. By default it only creates an archive without compression. Compression can be applied using utilities like gzip, bzip2, or xz along with tar to create compressed archive files such as `.tar.gz`, `.tar.bz2`, or `.tar.xz`.
