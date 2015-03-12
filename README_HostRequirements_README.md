![alt text](https://github.com/InterGenOS/build_001/blob/master/InterGenOS-2015-02-21-400x226.png "InterGen OSsD")


#**The InterGenOS Project**
---
  
Until the installer has been completed, the initial setup for the builds has to be done manually
The scripts provided make the core system build fairly automated, and scripts are being written
to automate the extended packages as well.  Keep checking back for updated scripts.

Your host system should have the following software with the minimum versions indicated. 
This should not be an issue for most modern Linux distributions. 

Also note that many distributions will place software headers into separate packages, often in the form of
**<package-name>-devel** or **<package-name>-dev**. Be sure to install those if your distribution provides them.

---

<center>This build of **InterGenOS** was completed on an **ArchLinux** host system, with all available _**devel**_ type packages for the requirements listed below 
installed.</center>

```
-Bash-3.2 (/bin/sh should be a symbolic or hard link to bash)
-Binutils-2.17 (Versions greater than 2.25 are not recommended as they have not been tested)
-Bison-2.3 (/usr/bin/yacc should be a link to bison or small script that executes bison)
-Bzip2-1.0.4
-Coreutils-6.9
-Diffutils-2.8.1
-Findutils-4.2.31
-Gawk-4.0.1 (/usr/bin/awk should be a link to gawk)
-GCC-4.5.x including the C++ compiler, g++
-Glibc-2.10.x
-Grep-2.5.1a
-Gzip-1.3.12
-Linux Kernel-3.5.x
-M4-1.4.10
-Make-3.81
-Patch-2.5.4
-Perl-5.8.8
-Sed-4.1.5
-Tar-1.18
-Xz-5.0.0
```

><center>Some libraries used by gcc can be in an inconsistent state that interferes with building some InterGenOS packages. To check this, look in /usr/lib and possibly /usr/lib64 
for libgmp.la, libmpfr.la, and libmpc.la.  Either all three should be present or absent, but not only one or two. If the problem exists on your system, either rename or delete the 
.la files or install the appropriate missing package. You can use the lib_check.sh script below to check your system.</center>


The additional _**req_check.sh**_ and _**lib_check.sh**_ scripts can be used to assist in verifying the requirements

```
#!/bin/bash
#req_check.sh - script to verify requirements of critical development tools

export LC_ALL=C
bash --version | head -n1 | cut -d" " -f2-4
echo "/bin/sh -> `readlink -f /bin/sh`"
echo -n "Binutils: "; ld --version | head -n1 | cut -d" " -f3-
bison --version | head -n1

if [ -h /usr/bin/yacc ]; then
  echo "/usr/bin/yacc -> `readlink -f /usr/bin/yacc`";
elif [ -x /usr/bin/yacc ]; then
  echo yacc is `/usr/bin/yacc --version | head -n1`
else
  echo "yacc not found" 
fi

bzip2 --version 2>&1 < /dev/null | head -n1 | cut -d" " -f1,6-
echo -n "Coreutils: "; chown --version | head -n1 | cut -d")" -f2
diff --version | head -n1
find --version | head -n1
gawk --version | head -n1

if [ -h /usr/bin/awk ]; then
  echo "/usr/bin/awk -> `readlink -f /usr/bin/awk`";
elif [ -x /usr/bin/awk ]; then
  echo yacc is `/usr/bin/awk --version | head -n1`
else 
  echo "awk not found" 
fi
                                                                                                                                                                                             
gcc --version | head -n1                                                                                                                                                                     
g++ --version | head -n1                                                                                                                                                                     
ldd --version | head -n1 | cut -d" " -f2-  # glibc version
grep --version | head -n1
gzip --version | head -n1
cat /proc/version
m4 --version | head -n1
make --version | head -n1
patch --version | head -n1
echo Perl `perl -V:version`
sed --version | head -n1
tar --version | head -n1
xz --version | head -n1

echo 'main(){}' > dummy.c && g++ -o dummy dummy.c
if [ -x dummy ]
  then echo "g++ compilation OK";
  else echo "g++ compilation failed"; fi
rm -f dummy.c dummy

#End req_check.sh
```

---

```
#!/bin/bash
#lib_check.sh - script to verify that library irregularities aren't present


for lib in lib{gmp,mpfr,mpc}.la; do
  echo $lib: $(if find /usr/lib* -name $lib|
               grep -q $lib;then :;else echo not;fi) found
done
unset lib

#End lib_check .sh
```

---

#<center>Partitions and Filesystems</center>

><center>These notes are to be used as a guide, so if you're not familiar with the content or terminology, don't just *run the commands*- ask questions first!  :)</center>

Use cfdisk or fdisk with a command line option, or a gui partitioning utility such as gparted, naming 
the hard disk on which the build partition will be created— for example /dev/sda for the primary disk. 
  
Create an ext4 Linux native partition no less than 40GB in size, and also create a small swap partition 
if the host machine has less than 6GB of physical RAM.

---

####CLI examples for setting up fs and swap
(if you're the 'have to do it manually-type)


#####Create build file system- CLI style
`mkfs -v -t ext4 /dev/<sdx>`

#####Create swap if needed- CLI style
`mkswap /dev/<sdy>`

###Choose a mount point and assign it to the InterGenOS environment variable
`export IGos=/mnt/igos`

 >Add the above listed export line to both your ~/.bash_profile, and your /root/.bash_profile so that it will be sourced into the environment at every boot.

###Verify the environment variable is in place
`echo $IGos`

###Expected return for 'echo $IGOs':
`/mnt/igos`

###Enable swap if needed- CLI style
`/sbin/swapon -v /dev/<sdy>`

---
##<center>Ready to build???</center>

###<center>Run the setup.sh script as user 'root'<center>
