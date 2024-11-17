## Networking between host and namespace

To connect the namespace created using `unshare` to the main Linux host machine using virtual Ethernet pairs, follow these steps:

---

### **Steps to Connect the Namespace to the Host Machine**

#### **1. Run the `unshare` Command**
The command creates the isolated environment. Ensure the environment is set up as expected:
```bash
sudo unshare --mount --uts --ipc --net --pid --fork --mount-proc chroot mycontainer /bin/bash
```

#### **2. Identify the Namespace Process**
The namespace created by `unshare` is associated with the process ID (`PID`) of the new shell. Identify the PID using:
```bash
ps aux | grep /bin/bash
```

- For a detail steps about how to find namespace's PID click [Find-PID-of-the-namespace](11_find_PID.md)

#### **3. Create a Virtual Ethernet Pair**
Create a virtual Ethernet (veth) pair to act as a link between the host and the namespace:
```bash
sudo ip link add veth-host type veth peer name veth-ns
```
- `veth-host`: One end of the veth pair that remains on the host.
- `veth-ns`: The other end that will be moved to the namespace.

#### **4. Move `veth-ns` to the Namespace**
Use the `ip link set` command to assign `veth-ns` to the namespace using the namespace's PID:
```bash
sudo ip link set veth-ns netns <namespace-pid>
```

Replace `<namespace-pid>` with the PID of the process running the namespace.

#### **5. Configure `veth-host` on the Host**
Assign an IP address to `veth-host` and bring it up:
```bash
sudo ip addr add 192.168.1.1/24 dev veth-host
sudo ip link set veth-host up
```

#### **6. Configure `veth-ns` Inside the Namespace**
Enter the namespace process (if you are not already inside):
```bash
sudo nsenter --net=/proc/<namespace-pid>/ns/net
```

Once inside, configure `veth-ns`:
```bash
ip addr add 192.168.1.2/24 dev veth-ns
ip link set veth-ns up
ip link set lo up
```

#### **7. Set Up Routes**
Inside the namespace, set the default route to point to the host:
```bash
ip route add default via 192.168.1.1
```

---

### **8. (Optional) Enable IP Forwarding and NAT on the Host**
To allow the namespace to communicate with external networks through the host, enable IP forwarding:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Set up NAT using `iptables`:
```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.1.2/24 -o <host-interface> -j MASQUERADE
```

Replace `<host-interface>` with your host's active network interface (e.g., `eth0` or `wlan0`).

---

### **9. Test the Connection**
- From the host, test connectivity to the namespace:
  ```bash
  ping 192.168.1.2
  ```
- From the namespace, test connectivity to the host:
  ```bash
  ping 192.168.1.1
  ```
- (Optional) To test external network connectivity from the namespace:
  ```bash
  ping 8.8.8.8
  ```

---

### **10. Cleanup**
After you're done, clean up the setup to avoid leaving residual configurations.

1. Remove the veth pair:
   ```bash
   sudo ip link del veth-host
   ```
2. Remove the NAT rule:
   ```bash
   sudo iptables -t nat -D POSTROUTING -s 192.168.1.2/24 -o <host-interface> -j MASQUERADE
   ```

---

### **Summary**
1. Use `unshare` to create the namespace.
2. Create a veth pair and assign one end to the host and the other to the namespace.
3. Configure IP addresses and routes for the veth interfaces.
4. Enable IP forwarding and set up NAT for external connectivity.

Let me know if you need further clarification!

