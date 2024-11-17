## Find PID of the namespace 

To note the **PID of the shell** running inside the namespace after executing `unshare`, you can use one of the following methods:

---

### **1. Use `ps` Command**
After running the `unshare` command, check the list of processes with `ps`. The innermost `bash` process is the one running inside the namespace.

For example:
```bash
sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
```

In another terminal, use `ps` to find the PID:
```bash
ps -ef | grep "[/]bin/bash"
```

This will return the PID of the `bash` shell.

---

### **2. Use `echo $$` in the Shell**
After running `unshare` and being inside the namespace shell, you can directly get its PID using:
```bash
echo $$
```

The `$$` variable holds the PID of the current shell process.

---

### **3. Use `pgrep` Command**
Use `pgrep` to find the PID of the `bash` shell:
```bash
pgrep -af /bin/bash
```

This will list all running `bash` processes. Look for the one associated with your namespace (e.g., `unshare`).

---

### **4. Use `lsns` Command**
The `lsns` command shows all active namespaces and their associated PIDs. You can filter by network namespace (`net`), for example:
```bash
lsns | grep net
```

This shows the namespace inode and the PID of the process holding that namespace.

---

### **5. Use `pstree`**
`pstree` shows a tree of processes. Look for your `unshare` command and the child `bash` process:
```bash
pstree -p | grep -A 3 unshare
```

This provides a clear hierarchy of processes, including the PID of the `bash` shell.

---

### **Example Workflow**
1. Run `unshare` in a terminal:
   ```bash
   sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
   ```

2. In another terminal, find the `bash` process using `ps`:
   ```bash
   ps af
   ```

   Example output:
   ```
   12360 pts/0    S+     0:00  \_ sudo unshare ...
   12361 pts/6    Ss     0:00      \_ sudo unshare ...
   12362 pts/6    S      0:00          \_ unshare ...
   12363 pts/6    S+     0:00              \_ /bin/bash
   ```

   The PID you need is `12363`.

---

### **Using the PID**
Once you have the PID of the namespace's `bash` shell, you can attach resources like `veth`:
```bash
sudo ip link set veth-ns netns <bash-PID>
```

Let me know if you need further assistance!