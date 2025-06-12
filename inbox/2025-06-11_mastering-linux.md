---
date: 2025-06-11
tags:
  - course
hubs:
  - "[[linux]]"
  - "[[dev]]"
urls:
  - udemy.com/course/mastering-linux
  - udemy.com/course/linux-foundation-certified-systems-administrator-lfcs
  - .udemy.com/course/unofficial-linux-redhat-certified-system-administrator-rhcsa
---

# mastering linux

## User Account Management

Commands:

- useradd: Doesn't create home folder by default
- adduser
- groupadd
- userdel
- groupdel
- usermod

Files:

- /etc/passwd
- /etc/group
- etc/shadow

1. Creating user

```bash
sudo useradd <username>
```

## Redirection - Manage Data Streams

Commands:

Files:

Operators:

- output redirection with ">" or "1>"
  - ">" or "1>" is used to redirect output to a file.
  - If the file doesn't exist, it willbe created.
  - Otherwise, the file will be overwritten.
- output redirection with ">>" or "1>>"
  - Used to append to a file.
  - If the file doesn't exist, we get error ": No such file or directory" & this
    error is not appended to the file.
- error redirection with "2>" or "2>>"

  - "2>" is used to redirect errors to a file.
  - to omit errors from the terminal in chained commands or scripts.

  ```bash
  # input and error redirection
  du -h file1.txt file2.txt 1> output.txt 2> error.txt

  # redirection or erro to /dev/null
  du -h file1.txt file2.txt 2> /dev/null
  ```

Concepts:

- Standard streams:  
  There are 3 communication channels for data:

      - 0: standard input or stdin
      - 1: standard output or stdout
      - 2: standard error or stderr

> [!NOTE] We can combine stream operators  
> Like error to a seperate file and output in a seperate one.
