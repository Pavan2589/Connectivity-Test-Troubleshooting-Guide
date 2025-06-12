# Team Shunya – Husarnet VPN ROS 2 Connectivity & Performance Troubleshooting Guide

This guide documents common connectivity issues and performance bottlenecks encountered during ROS 2 networking over **Husarnet VPN**, especially during remote testing and robot teleoperation. It also provides **tested fixes** for each problem.

---

## Recommended Setup Before Troubleshooting

Before diving into connectivity issues, make sure you've correctly configured the **Fast DDS Discovery Server**, which is essential for ROS 2 communication over VPN.

**Guide:** [Fast DDS Discovery Server Setup with Husarnet](https://husarnet.com/docs/ros2/ros-discovery-server-env/)

This setup ensures that nodes can discover each other reliably across remote systems connected via Husarnet.

---

##  Table of Contents

* [1. Husarnet Tunnel Instead of Direct Peer-to-Peer](#1-husarnet-tunnel-instead-of-direct-peer-to-peer)
* [2. Husarnet Join Hangs or Fails](#2-husarnet-join-hangs-or-fails)
* [3. Topics Not Visible After VPN & DDS Setup](#3-topics-not-visible-after-vpn--dds-setup)
* [4. UFW Firewall Restrictions](#4-ufw-firewall-restrictions)
* [5. Twist vs TwistStamped Message Mismatch](#5-twist-vs-twiststamped-message-mismatch)
* [6. Resolving Image Feed Issues with Uncompressed Topics](#6-resolving-image-feed-issues-with-uncompressed-topics)

  * [6.1. Reduce IP Fragment Timeout](#61-reduce-ip-fragment-timeout)
  * [6.2. Increase IP Fragment Memory Threshold](#62-increase-ip-fragment-memory-threshold)
  * [6.3. Increase Linux Receive Buffer Size](#63-increase-linux-receive-buffer-size)
* [7. Changing MTU Size](#7-changing-mtu-size)
* [8. Make Kernel Parameter Changes Permanent](#8-make-kernel-parameter-changes-permanent)
* [9. Validate the Settings](#9-validate-the-settings)

---

## 1. Husarnet Tunnel Instead of Direct Peer-to-Peer

If Husarnet shows "tunneled" instead of a peer-to-peer connection:

```bash
sudo husarnet restart
```

---

## 2. Husarnet Join Hangs or Fails

Husarnet web setup taking too long or join fails due to old config:

```bash
sudo systemctl stop husarnet
sudo rm -rf /etc/husarnet/*
sudo systemctl start husarnet
sudo husarnet join <your_join_code>
```

---

## 3. Topics Not Visible After VPN & DDS Setup

Ensure ROS 2 daemon is reset after Fast DDS Discovery is initiated:

```bash
ros2 daemon stop
ros2 daemon start
```

---

## 4. UFW Firewall Restrictions

Check UFW status. It should be inactive:

```bash
sudo ufw status
sudo ufw disable
```


## 5. Twist vs TwistStamped Message Mismatch

If your robot expects `geometry_msgs/msg/TwistStamped` instead of `Twist`, create a **bridge node** to convert messages:

> ✅ The bridge node is available in this repository under the `twist_stamped_bridge` package.

> It converts messages from `teleop_twist_keyboard` (`Twist`) to `TwistStamped`.


## 6. Resolving Image Feed Issues with Uncompressed Topics

### 6.1. Reduce IP Fragment Timeout

```bash
sudo sysctl -w net.ipv4.ipfrag_time=3
```

### 6.2. Increase IP Fragment Memory Threshold

```bash
sudo sysctl -w net.ipv4.ipfrag_high_thresh=134217728
```

### 6.3. Increase Linux Receive Buffer Size

```bash
sudo sysctl -w net.core.rmem_max=2147483647
```

---

## 7. Changing MTU Size

Increase MTU to handle large messages (e.g., image/video streaming):

```bash
sudo ip link set dev hnet0 mtu 9000
```

Make it permanent:

```bash
sudo nano /etc/netplan/<your_config_file>.yaml
```

Paste:

```yaml
network:
  version: 2
  ethernets:
    hnet0:
      dhcp4: yes
      mtu: 9000
```

Then apply:

```bash
sudo netplan apply
```

---

## 8. Make Kernel Parameter Changes Permanent

Create a config file:

```bash
sudo nano /etc/sysctl.d/10-cyclone-max.conf
```

Paste:

```bash
# IP fragmentation settings
net.ipv4.ipfrag_time=3
net.ipv4.ipfrag_high_thresh=134217728
# Increase the maximum receive buffer size for network packets
net.core.rmem_max=2147483647
```

Apply changes:

```bash
sudo sysctl --system
```

---

## 9. Validate the Settings

Ensure the new settings have been applied:

```bash
sysctl net.core.rmem_max net.ipv4.ipfrag_time net.ipv4.ipfrag_high_thresh
```

Expected Output:

```
net.core.rmem_max = 2147483647
net.ipv4.ipfrag_time = 3
net.ipv4.ipfrag_high_thresh = 134217728
```

---

## ✅ Summary

These fixes optimize Husarnet VPN and ROS 2 performance by ensuring:

* Clean VPN join operations
* Correct peer-to-peer routing
* Compatibility between message types
* Improved reliability of large data streams

**Sources**:
DDS Middleware and Network Tuning – Stereolabs
[https://www.stereolabs.com/docs/ros/networking/](https://www.stereolabs.com/docs/ros/networking/)

Husarnet Troubleshooting Guide
[https://husarnet.com/docs/troubleshooting-guide/](https://husarnet.com/docs/troubleshooting-guide/)



