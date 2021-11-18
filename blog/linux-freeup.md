# Free Up Space After Removing Log File in Linux

![](linux_fd.png)

There were so many times that I faced this situation where we simply  removing (rm command) log files while the service is still running. What is happening here? Well, the process still uses that file to write the  logs, that means the process won’t release the file until it gets the  signal like what rsyslog does.

Here is the example, I deleted a huge log file, let’s say, `/var/log/service/service.log`, now I figured out that the disk space did not change. That’s the main reason I mentioned earlier. 

```
narbeh@nsrv:~$ df -h
Filesystem             Size  Used Avail Use% Mounted on
udev                   3.7G     0  3.7G   0% /dev
tmpfs                  760M  9.7M  750M   2% /run
/dev/sda6               50G   44G    0G 100% /
tmpfs                  3.8G  573M  3.2G  16% /dev/shm
```

### Solution

Mostly you won’t be able to restart the service as it might be on a  production server or such, but how can we send the signal to the process to release the deleted file without restarting?

Let’s first list the deleted files in the system using `lsof +L1` command:

```
narbeh@nsrv:~$ lsof +L1
COMMAND     PID   USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
upstart    2014 narbeh   10w   REG   0,43      217     0 1311608 /home/narbeh/.cache/upstart/window-stack-bridge.log.1 (deleted)
upstart    2014 narbeh   27w   REG   0,43      271     0 1313368 /home/narbeh/.cache/upstart/indicator-application.log.1 (deleted)
upstart    2014 narbeh   30w   REG   0,43      127     0 1316997 /home/narbeh/.cache/upstart/indicator-printers.log.1 (deleted)
service    4992 narbeh   26u   REG   0,43   777123     0    1297 /var/log/service/service.log (deleted)
slack      2589 narbeh   64r   REG   0,21   131072     0      27 /dev/shm/.org.chromium.Chromium.UCc5KW (deleted)
slack      2589 narbeh   66u   REG   0,21        4     0      30 /dev/shm/.org.chromium.Chromium.h0CMmn (deleted)
slack      2589 narbeh   76u   REG   0,21      144     0      29 /dev/shm/.org.chromium.Chromium.gdhogO (deleted)
slack      2589 narbeh   98u   REG   0,21      144     0      34 /dev/shm/.org.chromium.Chromium.vGJjlf (deleted)
slack      2881 narbeh   22u   REG   0,21        4     0      30 /dev/shm/.org.chromium.Chromium.h0CMmn (deleted)
```

Find your deleted file name in the `NAME` column or grep it. You can see the `PID` which here for my service is `4992`. In addition, there is a column named FD which points to the number which you will find it under `/proc/PID/fd/FD`.

Go to `/proc/PID` and you will find a directory called `fd` there. FD stands for File Descriptor. 

```
narbeh@nsrv:~$ ls -lh /proc/4992/fd/26
lrwx------ 1 narbeh narbeh 64 Apr  9 22:22 /proc/4992/fd/26 -> /var/log/service/service.log (deleted)
```

Now we’ve got the main file descriptor’s path. We can now truncate it while the service is still using it:

```
narbeh@nsrv:~$ cat /dev/null > /proc/4992/fd/26
```

#### Prevention

If you are using `logrotate`, it will gracefully send the signal while rotating, but if you need to do it fast, just use the last command and `/dev/null` or simply run `echo > LOG_FILE`  instead of removing it.

Read more about [file descriptors](https://en.wikipedia.org/wiki/File_descriptor)