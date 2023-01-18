---
title: Open NFS ports
weight: 400
keywords: sharedv4 volumes, ReadWriteMany, PVC, kubernetes, k8s
description: Open the NFS ports required for sharedv4 volumes to work.
hidden: true
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/open-nfs-ports/
---
SharedV4 volumes utilize NFS services, and they therefore require specific open NFS ports to allow for communication between nodes in your cluster. Depending on how your cluster nodes are configured, your firewall may block some of these ports, or your NFS ports may differ from the defaults. To solve these issues, you may need to manually assign NFS ports and ensure that your firewall or ACL allows them to communicate.

This document provides instructions for detecting and opening NFS ports according to various cluster configurations that you may have.

## Prerequisites

All of the use-cases in this document will require that the mandatory Portworx network port ranges are open between nodes in the cluster, as documented [here](/install-portworx/prerequisites/).

## Determine which ports to open

First, check what the existing NFS port configuration is for your nodes to see if they need to be remapped.

Enter the following command to find which ports NFS is using on your node:

```text
rpcinfo -p
```

SharedV4 volumes communicate on the following standard ports/services:

* PortMapper: tcp/udp 111 (default on most Linux distributions)
* NFSd: tcp/udp 2049 (default on most Linux distributions)
* MountD: tcp/udp 20048 (**depends on the Linux distribution**)

If the ports listed from the above `rpcinfo` output on your nodes match these standard ports, proceed to [Open standard NFS ports (most Linux distributions)](#open-standard-nfs-ports-most-linux-distributions).

If the NFS ports on your OS do not match these ports, or your OS randomly chooses the ports for these services, proceed to [Manually assign and open NFS ports](#manually-assign-and-open-nfs-ports).

## Open standard NFS ports (most Linux distributions)

If your Linux distribution uses the standard ports identified in the previous section, you do not need to manually assign any ports for NFS, but you may need to open them.

Ensure that your ports are open on any firewalls and your ACL by entering the following commands:

```text
iptables -I INPUT -p tcp -m tcp --match multiport --dports 111,2049,20048 -j ACCEPT
iptables -I OUTPUT -p tcp -m tcp --match multiport --dports 111,2049,20048 -j ACCEPT
```

Once you've determined that your hosts are using the standard ports and that you have opened those ports, you can start using SharedV4 volumes.

## Manually assign and open NFS ports

For certain Linux distributions, the OS chooses the `mountd` port randomly every time the node reboots. To solve this, you must manually assign NFS ports, and how you accomplish this depends on your OS.

Only perform the steps in one of the following sections if one of the following is true:

* The `mountd` port is not fixed (and not the standard port of 20048) and is chosen at random by your Linux distribution.
* You wish to open a contiguous range of ports for Portworx and want to shift the default NFS ports to your Portworx port range.

In order to manually assign and open NFS ports, follow the steps in the section that applies for your OS.

### Assign NFS ports on the RedHat family of Linux (RHEL, CentOS, Fedora, etc)

1. Modify the `/etc/sysconfig/nfs` file, uncommenting or adding the following fields and assigning the associated values:

    * LOCKD_TCPPORT=9023
    * LOCKD_UDPPORT=9024
    * MOUNTD_PORT=9025
    * STATD_PORT=9026

2. Enter the following command to restart the NFS server:

    ```text
    systemctl restart nfs-server
    ```

3. Open the newly assigned NFS ports on your access control list:

    ```text
    iptables -I INPUT -p tcp -m tcp --match multiport --dports 111,2049,9023,9025,9026 -j ACCEPT
    iptables -I OUTPUT -p tcp -m tcp --match multiport --dports 111,2049,9023,9025,9026 -j ACCEPT
    iptables -I INPUT -p udp -m udp --dport 9024  -j ACCEPT
    iptables -I OUTPUT -p udp -m udp --dport 9024  -j ACCEPT
    ```

### Open NFS ports on Debian or Ubuntu Linux

#### Debian 10 / Ubuntu

1. Modify the `/etc/default/nfs-kernel-server` file. Uncomment or add the `RPCMOUNTDARGS` field and add the `--port 9024` option to the value:
   
   ```text
   ...
   RPCMOUNTDOPTS="--manage-gids --port 9024"
   ```

2. Enter the following command to restart the `mountd` service:

    ```text
    systemctl restart nfs-mountd.service
    ```

3. Verify that the `mountd` service is running on the port that you configured by searching the output of `rcpinfo -p`:
   
    ```text
    rpcinfo -p | grep 'tcp.*mountd'
    ```
    ```output
    100005    1   tcp   9024  mountd
    100005    2   tcp   9024  mountd
    100005    3   tcp   9024  mountd
    ```

#### Debian 9 and lower

1. Modify the `/run/sysconfig/nfs-utils` file, uncommenting or adding the following fields and assigning the associated values:

    * `RPCNFSDARGS=" 8 --port 9023"`: append the `--port 9023` option to any existing values.
    * `RPCMOUNTDARGS="--port 9024"`: add the `--port 9024` option.
    * `STATDARGS="--port 9025 --outgoing-port 9026"`: add the `--port 9025` and `--outgoing-port 9026` options.

2. Enter the following commands to restart the NFS server:

    ```text
    systemctl daemon-reload
    systemctl restart rpc-statd
    systemctl restart rpc-mountd
    systemctl restart nfs-server
    ```

3. Open the newly assigned NFS ports on your access control list:

    ```text
    iptables -I INPUT -p tcp -m tcp --match multiport --dports 111,2049,9023,9025,9026 -j ACCEPT
    iptables -I OUTPUT -p tcp -m tcp --match multiport --dports 111,2049,9023,9025,9026 -j ACCEPT
    iptables -I INPUT -p udp -m udp --dport 9024  -j ACCEPT
    iptables -I OUTPUT -p udp -m udp --dport 9024  -j ACCEPT
    ```

### Open NFS ports on CoreOS

The following sharedv4 NFS services run on a node when Portworx is installed with sharedv4 support:

* portmapper
* status
* mountd
* nfs and nfs_acl
* nlockmgr

View these services and the ports they are using by entering the following command:

    ```text
    rcpinfo -p
    ```

By default, services like `mountd` and `nlockmgr` run on random ports and must be fixed to a specific port.

#### Configure nfs, nfs_acl, and lockd ports

By default, the `nfs-server.service` configuration file is located under the following directory:

```text
/usr/lib/systemd/system/nfs-server.service
```

1. Check the status of the existing `nfs-server.service`:

    ```text
    systemctl status nfs-server
    ```

2. Copy the systemd unit file from the `usr` directory into the `etc` directory:

    ```text
    cp /usr/lib/systemd/system/nfs-server.service /etc/systemd/system/nfs-server.service
    ```

3. Open `/etc/systemd/system/nfs-server.service` in a text editor and, under the `[Service]` section, add the `--port 9023` value to the `ExecStart=/usr/sbin/rpc.nfsd` key:

    ```text
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStartPre=/usr/sbin/exportfs -r
    ExecStart=/usr/sbin/rpc.nfsd --port 9023
    ExecStop=/usr/sbin/rpc.nfsd 0
    ExecStopPost=/usr/sbin/exportfs -au
    ExecStopPost=/usr/sbin/exportfs -f
    ```

4. Update the `lockd` ports:

    ```text
    echo 9027 > /proc/sys/fs/nfs/nlm_udpport
    echo 9028 > /proc/sys/fs/nfs/nlm_tcpport
    ```

5. Ensure that the `lockd` manager ports persist over node reboots by creating a `100-nfs-ports.conf` file under the `/etc/sysctl.d/` folder and adding the ports to it:

    ```text
    cat /etc/sysctl.d/100-nfs-ports.conf
    fs.nfs.nlm_tcpport = 9027
    fs.nfs.nlm_udpport = 9028
    ```

6. Reload the `systemd` daemon and restart the `nfs-server` service:

    ```text
    systemctl daemon-reload
    systemctl restart nfs-server
    ```

7. Verify that the NFS services are running on the ports you configured by searching the output of `rcpinfo -p`:

    ```text
    rpcinfo -p | grep nfs
    ```
    ```output
    100003    3   tcp   9023  nfs
    100003    4   tcp   9023  nfs
    100227    3   tcp   9023  nfs_acl
    ```

    ```text
    rpcinfo -p | grep nlock
    ```
    ```output
    100021    1   udp   9027  nlockmgr
    100021    3   udp   9027  nlockmgr
    100021    4   udp   9027  nlockmgr
    100021    1   tcp   9028  nlockmgr
    100021    3   tcp   9028  nlockmgr
    100021    4   tcp   9028  nlockmgr
    ```

#### Configure mountd services

1. Check the status of the existing `nfs-server.service`:

    ```text
    systemctl status nfs-mountd
    ```

2. Copy the systemd unit file from `/usr` into `/etc`:

    ```text
    cp /usr/lib/systemd/system/nfs-mountd.service /etc/systemd/system/nfs-mountd.service
    ```

3. Open `/etc/systemd/system/nfs-mountd.service` in a text editor and, under the `[Service]` section, add the `--port 9024` value to the `ExecStart=/usr/sbin/rpc.mountd` key:

    ```text
    [Unit]
    ...

    [Service]
    Type=forking
    ExecStart=/usr/sbin/rpc.mountd --port 9024
    ```

4. Reload the `systemd` daemon and restart the `nfs-server` service:

    ```text
    systemctl daemon-reload
    systemctl restart nfs-server
    ```

5. Verify that the NFS services are running on the ports you configured by searching the output of `rcpinfo -p`:

    ```text
    rpcinfo -p | grep mountd
    ```
    ```output
    100005    1   udp   9024  mountd
    100005    1   tcp   9024  mountd
    100005    2   udp   9024  mountd
    100005    2   tcp   9024  mountd
    100005    3   udp   9024  mountd
    100005    3   tcp   9024  mountd
    ```

#### Configure statd services

1. Check the status of the existing `rpc-statd.service`:

    ```text
    systemctl status rpc-statd
    ```
    <!-- what should users be looking for here? -->

2. Copy the systemd unit file from the `usr` directory into the `etc` directory:

    ```text
    cp /usr/lib/systemd/system/rpc-statd.service /etc/systemd/system/rpc-statd.service
    ```

3. Open `/etc/systemd/system/rpc-statd.service` in a text editor and, under the `[Service]` section, add the `--port 9025` and `--outgoing-port 9026` value to the `ExecStart=/usr/sbin/rpc.statd` key:

    ```text
    [Unit]
    ...

    [Service]
    Environment=RPC_STATD_NO_NOTIFY=1
    Type=forking
    PIDFile=/var/run/rpc.statd.pid
    ExecStart=/usr/sbin/rpc.statd --port 9025 --outgoing-port 9026
    ```

4. Reload the `systemd` daemon and restart the `rpc-statd` service:

    ```text
    systemctl daemon-reload
    systemctl restart rpc-statd
    ```

5. Verify that the NFS services are running on the ports you configured by searching the output of `rcpinfo -p`:

    ```text
    rpcinfo -p | grep status
    ```
    ```output
    100024    1   udp   9025  status
    100024    1   tcp   9025  status
    ```

### Open NFS ports on PhotonOS

1. Modify the `/etc/default/nfs-utils` file, adding parameters to assign ports to services:

    ```text
    MOUNTD_OPTS="--port 9024"
    NFSD_OPTS="--port 9023"
    STATD_OPTS="--port 9025 --outgoing-port 9026"
    ```

2. Add the `/etc/modprobe.d/lockd.conf` file to assign ports to the `nlockmgr` service and reconfigure runtime ports:

    ```text
    cat > /etc/modprobe.d/lockd.conf << .
    options lockd nlm_tcpport=9027
    options lockd nlm_udpport=9028
    .

    sysctl -w fs.nfs.nlm_tcpport=9027 fs.nfs.nlm_udpport=9028
    ```

3. Restart NFS services:

    ```text
    systemctl daemon-reload
    systemctl restart nfs-server nfs-mountd rpc-statd
    ```

4. Open up the ports 9001 to 9028 in the `iptables` firewall, then restart the service:

    ```text
    # add Portworx ports 9001..9028 before "COMMIT"
    sed -i~ '/^COMMIT/i -A INPUT -p tcp -m multiport --dports 111,9001:9028 -j ACCEPT' \
      /etc/systemd/scripts/ip4save
    systemctl restart iptables
    ```

5. Verify that the NFS services are running on the ports that you just configured:

    ```text
    rpcinfo -p
    ```
    ```output
    program vers proto   port  service
     100000    4   tcp    111  portmapper
     100000    3   tcp    111  portmapper
     100000    2   tcp    111  portmapper
     100000    4   udp    111  portmapper
     100000    3   udp    111  portmapper
     100000    2   udp    111  portmapper
     100005    1   udp   9024  mountd
     100005    1   tcp   9024  mountd
     100005    2   udp   9024  mountd
     100005    2   tcp   9024  mountd
     100005    3   udp   9024  mountd
     100005    3   tcp   9024  mountd
     100024    1   udp   9025  status
     100024    1   tcp   9025  status
     100003    3   tcp   9023  nfs
     100003    4   tcp   9023  nfs
     100021    1   udp   9027  nlockmgr
     100021    3   udp   9027  nlockmgr
     100021    4   udp   9027  nlockmgr
     100021    1   tcp   9028  nlockmgr
     100021    3   tcp   9028  nlockmgr
     100021    4   tcp   9028  nlockmgr
    ```
