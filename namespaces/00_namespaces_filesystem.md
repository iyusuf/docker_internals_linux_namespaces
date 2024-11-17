# Namespaces and Isolated Filesystem



## 1. Lab 

Can I simulate a Docker container image and running it using raw Linux namespaces? If yes, can you show me step by step of a image that will run a shell script hello world?

### ***Layer 1*** Create a Linux root filesystem 

- Create a root folder
```bash
cd namespaces
mkdir -p mycontainer/bin
```

- Copy essential binaries
```bash
cp /bin/bash mycontainer/bin/
cp /bin/echo mycontainer/bin/
```

- ldd is a command-line utility in Linux that lists the shared libraries required by a given program or shared object
```bash
ldd /bin/bash
        linux-vdso.so.1 (0x00007ffdfb3f7000)
        libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007feed9634000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007feed940b000)
        /lib64/ld-linux-x86-64.so.2 (0x00007feed97cf000)
```

- Copy shared libraries from ldd output.
```bash
sudo cp --parents /lib/x86_64-linux-gnu/libtinfo.so.6 mycontainer/
sudo cp --parents /lib/x86_64-linux-gnu/libc.so.6 mycontainer/
sudo cp --parents /lib64/ld-linux-x86-64.so.2 mycontainer/
```

- Create the Shell Script
Place the hello.sh script inside the simulated container's filesystem.

- Copy code
```bash
mkdir -p mycontainer/scripts
echo -e '#!/bin/bash\necho "Hello, World!"' > mycontainer/scripts/hello.sh
chmod +x mycontainer/scripts/hello.sh
```

- Use unshare to create isolated namespaces and chroot to change the root directory. You will be inside a new shell session within the isolated environment. To check the PID of the current shell process, you can use the ps command within this new shell session.

```bash
sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
bash-5.1# /scripts/hello.sh
Hello, World!
bash-5.1#
```



### ***Layer 2*** Add Linux "ls" command  

- Copy 'ls'
```bash
cp --parent /bin/ls mycontainer/
```

- Find shared binaries that are needed for /bin/ls
```bash
ldd /bin/ls
        linux-vdso.so.1 (0x00007fff5ca47000)
        libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007faf99e36000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007faf99c0d000)
        libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007faf99b76000)
        /lib64/ld-linux-x86-64.so.2 (0x00007faf99e8e000)
```

- Copy shared files by finding files and files location from ldd output.
```bash
sudo cp --parents /lib/x86_64-linux-gnu/libselinux.so.1 mycontainer/
sudo cp --parents /lib/x86_64-linux-gnu/libc.so.6 mycontainer/
sudo cp --parents /lib/x86_64-linux-gnu/libpcre2-8.so.0 mycontainer/
sudo cp --parents /lib64/ld-linux-x86-64.so.2 mycontainer/
```

- Use 'unshare' command again to get into the isolated process. Now 'ls' should work in our container like isolated process.
```bash
sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
bash-5.1# ls
bin  lib  lib64  scripts
bash-5.1#
```

--- 

### ***Layer 3*** Add Linux "ip" and "ping" commands  

- Copy ip and ping to mycontainer
```bash
cp --parent /bin/ip mycontainer/
cp --parent /bin/ping mycontainer/

```

- Find and copy shared binaries using ldd for 'ip' and 'ping'
```bash
ldd /bin/ip
        linux-vdso.so.1 (0x00007ffe43bfb000)
        libbpf.so.0 => /lib/x86_64-linux-gnu/libbpf.so.0 (0x00007f95974c5000)
        libelf.so.1 => /lib/x86_64-linux-gnu/libelf.so.1 (0x00007f95974a7000)
        libmnl.so.0 => /lib/x86_64-linux-gnu/libmnl.so.0 (0x00007f959749f000)
        libbsd.so.0 => /lib/x86_64-linux-gnu/libbsd.so.0 (0x00007f9597487000)
        libcap.so.2 => /lib/x86_64-linux-gnu/libcap.so.2 (0x00007f959747c000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9597253000)
        libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f9597235000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9597617000)
        libmd.so.0 => /lib/x86_64-linux-gnu/libmd.so.0 (0x00007f9597228000)
ldd /bin/ping
        linux-vdso.so.1 (0x00007fff7e8c9000)
        libcap.so.2 => /lib/x86_64-linux-gnu/libcap.so.2 (0x00007f697eb19000)
        libidn2.so.0 => /lib/x86_64-linux-gnu/libidn2.so.0 (0x00007f697eaf8000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f697e8cf000)
        libunistring.so.2 => /lib/x86_64-linux-gnu/libunistring.so.2 (0x00007f697e725000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f697eb40000)
```

- Copy shared libraries
```bash

        sudo cp --parents /lib/x86_64-linux-gnu/libbpf.so.0  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libelf.so.1  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libmnl.so.0  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libbsd.so.0  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libcap.so.2  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libc.so.6 mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libz.so.1  mycontainer/
        sudo cp --parents /lib64/ld-linux-x86-64.so.2  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libmd.so.0  mycontainer/

        sudo cp --parents /lib/x86_64-linux-gnu/libcap.so.2  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libidn2.so.0  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libc.so.6  mycontainer/
        sudo cp --parents /lib/x86_64-linux-gnu/libunistring.so.2  mycontainer/
        sudo cp --parents /lib64/ld-linux-x86-64.so.2  mycontainer/
```


- Use 'unshare' command again to get into the isolated process. Now 'ip' and 'ping' should work in our container like isolated process.
```bash
sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
```



#### For now this is the end of adding layers 

### (Optional) Networking in namespaces
To continue with [networking with namespaces click me](10_namespace-networking.md)