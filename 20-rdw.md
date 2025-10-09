---
title: Using the Research Data Warehouse
teaching: 15
exercises: 15
---



entire episode to be written, content has been copied from episode 16-transferring-files.Rmd 

::::::::::::::::::::::::::::::::::::::: objectives

- Understand how to use Newcastle University's Research Data Warehouse (aka, RDW and Campus Filestore) with Comet HPC

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do I transfer files to (and from) the cluster?

::::::::::::::::::::::::::::::::::::::::::::::::::


## Transferring files to and from Campus Storage for Research Data (RDW)
RDW is mounted on Comet at `/rdw`.  Although it's a separate physical system it's 
located in the same data centre as Comet and connected via fast ethernet.
You can use scp and rsync to transfer data to RDW in the same way as copying to any other directory on Comet.  
RDW is intended for data storage and NOT suitable for interactive use or software installation.  
Working data should be in your home or project directory.
User installed software should be in your home directory

### Using cp to copy to RDW
Because `/rdw` is a mounted filesystem, we can use `cp` instead of `scp`:

```bash
[yourUsername@cometlogin01(comet) ~] cp file.txt /rdw/03/rse-hpc/training/userid/
[yourUsername@cometlogin01(comet) ~] cd /rdw/03/rse-hpc/training/userid/
[yourUsername@cometlogin01(comet) ~] pwd
```
```output
/rdw/03/rse-hpc/training/userid
```
```bash
[yourUsername@cometlogin01(comet) ~] ls
```
```output
file.txt
```

### Using rsync to copy to RDW

As you gain experience with transferring files, you may find the `scp`
command limiting. The [rsync](https://rsync.samba.org/) utility provides
advanced features for file transfer and is typically faster compared to both
`scp` and `sftp` (see below). It is especially useful for transferring large
and/or many files and creating synced backup folders.
The syntax is similar to `cp` and `scp`.  Rsync can be used on a locally mounted filesystem or a remote filesystem.

Transfer *to* RDW from your work area on Comet

#### Try out a dry run:
```bash
[yourUsername@cometlogin01(comet) ~] cd /nobackup/proj/training/userid/
[yourUsername@cometlogin01(comet) ~] mkdir TestDir
[yourUsername@cometlogin01(comet) ~] touch TestDir/testfile1
[yourUsername@cometlogin01(comet) ~] touch TestDir/testfile2
[yourUsername@cometlogin01(comet) ~] rsync -av TestDir /rdw/03/rse-hpc/training/userid --dry-run
```
```output
sending incremental file list
TestDir/
TestDir/testfile1
TestDir/testfile2

sent 121 bytes  received 26 bytes  294.00 bytes/sec
total size is 0  speedup is 0.00 (DRY RUN)
```

#### Run ‘for real’:
```bash
[yourUsername@cometlogin01(comet) ~] rsync -av TestDir /rdw/03/rse-hpc/training/userid
```
```output
sending incremental file list
created directory /rdw/03/rse-hpc/training/userid
rsync: chgrp "/rdw/03/rse-hpc/training/userid/TestDir" failed: Invalid argument (22)
TestDir/
TestDir/testfile1
TestDir/testfile2
rsync: chgrp "/rdw/03/rse-hpc/training/userid/TestDir/.testfile1.ofeRqX" failed: Invalid argument (22)
rsync: chgrp "/rdw/03/rse-hpc/training/userid/TestDir/.testfile2.fS1m6j" failed: Invalid argument (22)

sent 197 bytes  received 415 bytes  408.00 bytes/sec
total size is 0  speedup is 0.00
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1179) [sender=3.1.2]
```
What happened?  `rsync` returned an error. `files/attrs were not transferred `  This is because RDW doesn't 'know' about Comet's groups.  The transfer was successful though! Only the 'group' attribute of the file couldn't be transferred.  RDW has 'trumped' our local permissions and imposed its own standard permissions.  This isn't important, the correct user keeps ownership of the files.

```bash
[yourUsername@cometlogin01(comet) ~] ls -l TestDir/
```
```output
total 0
-rw------- 1 userid comet_training 0 Mar 11 20:06 testfile1
-rw------- 1 userid comet_training 0 Mar 11 20:06 testfile2

```
```bash
[yourUsername@cometlogin01(comet) ~] ls -l /rdw/03/rse-hpc/training/userid/TestDir/
```
```output
total 33
-rwxrwx--- 1 userid domainusers 0 Mar 11 20:10 testfile1
-rwxrwx--- 1 userid domainusers 0 Mar 11 20:10 testfile2
```
It’s still easier to read output without errors that we have to ignore, so let’s remove that error.

The `-a` (archive) option preserves permissions, this is why we see group modification errors above.  
For Comet and RDW, replace `-av` with `-rltv`  
`-r` = recurse through subdirectories    
`-l` = copy symlinks    
`-t` = preserve timestamps   
`-v` = verbose    


```bash
[yourUsername@cometlogin01(comet) ~] rsync -rltv TestDir/ /rdw/03/rse-hpc/training/userid 
```
```output
sending incremental file list
./
testfile1
testfile2

sent 150 bytes  received 57 bytes  414.00 bytes/sec
total size is 0  speedup is 0.00
```

:::challenge
## Spot the difference
Can you spot the difference betweent the 2 previous rsync commands?  Try `ls -l` on the destination.

:::solution
```bash
[yourUsername@cometlogin01(comet) ~] ls -R /rdw/03/rse-hpc/training/userid/
```
```output
/rdw/03/rse-hpc/training/userid/:
TestDir  testfile1  testfile2

/rdw/03/rse-hpc/training/userid/TestDir:
testfile1  testfile2

```

We now have too many files!  The first rsync command copied `TestDir` because there was no trailing `/`.   
The second rsync command only copied the contents of `TestDir` because of the trailing `/`.  
We could have spotted this by looking at the output of `--dry-run` but this shows it's a good idea to check the destination after you copy.

:::
:::


## Large data copies
When copying large amounts of data, rsync really comes into its own. When you're copying a lot of data, it's important to keep track in case the copy is interrupted.  Rsync is great because it can pick up where it left off, rather than starting the copy all over again.  It's also useful to output to a log so you can see what was transferred and find any errors that need to be addressed.

### Fast Connections
Transfers from Comet to RDW don’t leave our fast data centre network.  If you're using rsync with a fast network or disk to disk in the same machine:

- DON'T use compression `-z`
- DO use `--inplace`

Why?  compression uses lots of CPU, Rsync usually creates a temp file on disk before copying.  For fast transfers, this places too much load on the CPU and hard drive.    
`--inplace` tells rsync not to create the temp file but send the data straight away.  It doesn’t matter if the connection is interrupted, because rsync keeps track and tries again.  Always re-run transfer command to ensure nothing was missed.  The second run should be very fast, just listing all the files and not copying anything.

### Slow Connections
For a slow connection like the internet:
 
- DO use compression `-z` 
- DON’T use `--inplace`.  


:::challenge
##  Large Transfer to RDW
RDW has a super-fast connection to Comet, which means that it takes more resource to compress and un-compress the data than it does to do the transfer.
What command would best for backing up a large amount of data from Comet to RDW?

:::solution
```bash
[userid@login01 ~]$ rsync -rltv testDir/ /rdw/03/rse-hpc/training/userid
```

The `-a` option preserves permissions, this is why we saw group modification errors above.  For Comet and RDW, replace `-av` with `-rltv`    
`-r` = recurse through subdirectories    
`-l` = copy symlinks    
`-t` = preserve timestamps   
`-v` = verbose    
:::
:::

:::challenge
## add a dry run and a log file
 
:::solution
Try out a dry run:
```bash
rsync --dry-run -rltv --inplace --itemize-changes --progress --stats --whole-file --size-only /nobackup/myusername/source /rdw/path/to/my/share/destination/ 2>&1 | tee /home/myusername/meaningful-log-name.log1
```
Run ‘for real’:
```bash
rsync -rltv --inplace --itemize-changes --progress --stats --whole-file --size-only /nobackup/myusername/source /rdw/path/to/my/share/destination/ 2>&1 | tee /home/myusername/meaningful-log-name.log2
```

- `--inplace --whole-file --size-only` speed up transfer and prevent rsync filling up space with a large temporary directory    
- `--itemize-changes --progress --stats` for more informative output    
- Remember `|` from the Unix Shell workshop?    
 `| tee` sends output both to the screen and to a log file    
- All the arguments can be single letters like `-v` or full words like `--verbose`. Use `man rsync` to craft your favourite arguments list.

:::
:::


:::::::::::::::::::::::::::::::::::::::: keypoints

- `cp` and `rsync` transfer files between RDW and HPC.
- try a dry-run of rsync to avoid accidental duplications or deletions
- re-run large rsync commands to confirm success
- output to a log to keep a record
- group permissions are pre-set on RDW can't be changed from linux
- RDW shares should have a pre-set 'read' and 'modify' group of campus users
- ?? files on RDW are owned by the user who puts them there

::::::::::::::::::::::::::::::::::::::::::::::::::
