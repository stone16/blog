---
title: lsof/ ulimit/ ps
date: 2021-01-24 10:15:47
categories:
tags:
top:
---
# 1. lsof

# 1.1 File Descriptor in Linux

- Linux consider everything as a file
    - pipes, sockets, directories, devices, etc

# 1.2 What does lsof do?

- lsof means — List Open Files
- some columns need to understand
    - FD
        - stands for file descriptor
        - possible values
            - cwd — current working directory
            - rtd — root directory
            - txt — program text
            - mem — memory mapped file
            - (number)(parameter)
                - r for read acccess
                - w for write
                - u for both read and write
    - TYPE
        - DIR — directory
        - REG — Regular file
        - CHR — Character special file
        - FIFO — First In First Out

```jsx
// show long listing of open files, show cols like Command, PID, USER, FD, TYPE
# lsof 

// display the list of all opened file of uder leilei
# lsof -u leilei

// find process running on specific port
# lsof -i TCP:22

// list only IPv4 & IPv6 Open Files 
# lsof -i 4
# lsof -i 6

// list open files of TCP port ranges from 11 - 1023
# lsof -i TCP:1-1023

// exclude user with ^
# lsof -i -u^leilei

// list all network connections 
# lsof -i

// search by PID 
# lsof -p PID 

// kill all activities of particular user 
# kill -9 `lsof -t -u leilei`
```
# 2. ulimit

# 2.1 Overview

- Set or report the resource limit of the current user.
- Use with ulimit requires admin access, it only work on systems that allow control through the shell
- Types of resource limitation
    - hard limit
        - define the physical limit that the user can reach
    - soft limit
        - manageable by the user, its value can go up to the hard limit
- system resources are defined in a configuration file located at `/etc/security/limits.conf`

# 2.2 Common Commands

```jsx
// print all the resource limits for the current user 
# ulimit -a

// check the value of max core file size 
# ulimit -c 

// check the max data seg size
# ulimit -d

// check the max stack size of current user 
# ulimit -s

// check the max number of user processes 
# ulimit -u

// check the max number of threads 
# ulimit -T 

// check the size of virtual memory 
# ulimit -v 

// check time each process is allowed to run for 
# ulimit -t 

// check how many file descriptors a process can have 
# ulimit -n 
```
# 3. ps

# 3.1 Overview

- Linux is a multi-tasking and multi-user system
- so it allows multiple processes to operate simultaneously without interfering with each other
- A process is an executing instance of a program and carry out different tasks within the operating system

- PS command help us to review information related with the processes on a system
    - used to list the currently running processes and their PIDs along with some other information depends on different options

# 3.2 Some common commands

- Some common columns you should know what it means
    - PID - the unique process ID
    - TTY - terminal type that the user is logged into
    - TIME - amount of CPU in minutes and seconds that the process has been running
        - sometimes you see TIME as 00:00:00, merely means the total accumulated CPU utilization time for any process currently is 0
    - CMD - name of the command that launched the process
    - C - the CPU utilization in percentage
    - STIME - the start time of the process

```java
# ps 
PID TTY          TIME CMD
12330 pts/0    00:00:00 bash
21621 pts/0    00:00:00 ps

// view all the running processes 
# ps -A 
# ps -e

// view all processes associated with the terminal 
# ps -T 

// view all the running processes 
# ps -r 

// view all the processes owned by you 
# ps -x 

// print all the processes within the system 
# ps -e

// More detailed output by using -f option 
# ps -e -f 

// Search for a particular process 
# ps -C systemd
```

# Reference

[https://www.geeksforgeeks.org/lsof-command-in-linux-with-examples/](https://www.geeksforgeeks.org/lsof-command-in-linux-with-examples/)  

[https://www.tecmint.com/10-lsof-command-examples-in-linux/](https://www.tecmint.com/10-lsof-command-examples-in-linux/)