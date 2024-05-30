**Configure Mikrotik Router in a Ubuntu VM**

Create a VM using Mikrotik Router OS

**Update Two Mikrotik Router:**

Ensure you have the latest package lists:
Using Winbox:

Connect to your MikroTik router using Winbox.
Go to System > Packages.
Click the Check for Updates button.
If an update is available, you'll see the latest version listed. Choose the update channel (usually "stable") and click Download & Install.
The update will be downloaded and your router will reboot to complete the installation.

**Scenerio:** Let's asssume I have two mikrotik router server two different VPC and I want to connect two microtik router through Public ip using IPSec. VPC- Default route Public IP 111.119.218.65 and Private ip 192.168.1.84. VPC-A router Private IP 10.0.0.4 and Public ip 166.108.231.62 

**Router 1 (Public IP: 111.119.218.65, Private IP: 192.168.1.84):**

Log into Router 1 using Winbox.

IPSec Peer Configuration:

Go to IP > IPsec > Peers.
Click on the + button to add a new peer.
Set the Address to 166.108.231.62 (Public IP of Router 2).
Set Profile ( Example: ipsec-2)
Exchange Mode: IKE2
Leave the rest of the settings as default unless you need specific configurations.
Click Apply.
Click OK.

IPSec Policy Configuration:

Go to IP > IPsec > Policies.
Click on the + button to add a new policy.
General:
Peer set(Example: ipsec-2)
Click on Tunnel
Set Src. Address to 192.168.1.0/24 (assuming you want to allow the entire subnet).
Set Dst. Address to 10.0.0.0/24.
Protocol: 255
Action:
Action: Encrypt
Level: require
IPsec Protocols: esp
Proposal: set a name
Leave the rest as default, or adjust according to your requirements.
Click Apply.
Click OK.

IPSec Proposal Configuration:

Go to IP > IPsec > Proposals.
You can use the default proposal or create a new one.
Ensure the proposal is set to use esp, and configure encryption algorithms (e.g., aes-256 cbc and sha512).
Click Apply.
Click OK.

IPSec Identity Configuration:

Go to IP > IPsec > Identities.
Click the + button to add a new identity.
Set Peer to the peer created in step 2.
Set Auth. Method to pre-shared-key.
Enter the Secret (the same pre-shared key must be used on both routers).
Click OK.

**Router 2 (Public IP: 166.108.231.62, Private IP: 10.0.0.4):**

Log into Router 2 using Winbox.

IPSec Peer Configuration:

Go to IP > IPsec > Peers.
Click on the + button to add a new peer.
Set the Address to 111.119.218.65 (Public IP of Router 1).
Set Profile ( Example: ipsec-2)
Exchange Mode: IKE2
Leave the rest of the settings as default unless you need specific configurations.
Click Apply.
Click OK.

IPSec Policy Configuration:

Go to IP > IPsec > Policies.
Click on the + button to add a new policy.
General:
Peer set(Example: ipsec-2)
Click on Tunnel
Set Src. Address to 10.0.0.0/24.
Set Dst. Address to 192.168.1.0/24.
Leave the rest as default, or adjust according to your requirements.
Protocol: 255

Action:
Action: Encrypt
Level: require
IPsec Protocols: esp
Proposal: set a name
Leave the rest as default, or adjust according to your requirements.
Click Apply.
Click OK.

IPSec Proposal Configuration:

Go to IP > IPsec > Proposals.
You can use the default proposal or create a new one.
Ensure the proposal is set to use esp, and configure encryption algorithms (e.g., aes-256 cbc and sha512).
Click Apply.
Click OK.

IPSec Identity Configuration:

Go to IP > IPsec > Identities.
Click the + button to add a new identity.
Set Peer to the peer created in step 2.
Set Auth. Method to pre-shared-key.
Enter the Secret (the same pre-shared key must be used on both routers).
Click OK.

**Verify the Configuration:**
Check the IPSec Status:

Go to IP > IPsec > Installed SAs on both routers to see if the Security Associations (SAs) are established.
Ping Test:

From a device in the 192.168.1.0/24 subnet, ping a device in the 10.0.0.0/24 subnet to verify connectivity.
Similarly, from a device in the 10.0.0.0/24 subnet, ping a device in the 192.168.1.0/24 subnet.
Additional Configuration (if needed)
Firewall Rules:

Ensure that the firewall rules allow IPSec traffic (UDP port 500, ESP protocol).
Go to IP > Firewall > Filter Rules and NAT Rules to adjust any rules that might be blocking IPSec traffic.




**Make a directory to create the path**

```
sudo mkdir -p /mnt/nfs_share
```

---

Change the ownership for new group and new user for the directory `/mnt/nfs_share`

```
sudo chmod 777 /mnt/nfs_share
```

```
sudo chown -R nobody:nogroup /mnt/nfs_share
```

---

Then launch editor to  edit `/etc/exports` file for modifying the system configuration

On your ubuntu system, the NFS server is configured via the `/etc/exports` file. It details the directories you wish to share and the permissions for other devices on the network to access those directories.

```
sudo vim /etc/exports
```

---

After executing editor , we've to give the path of the directory where we want to get the access to read/write and sync the nfs server.You can configure the directories to be exported by adding them to the /etc/exports file. For example:

```
#/mnt/nfs_share 192.168.0.0/20 (rw,sync,no_subtree_check)   
```

---

 Apply the new config via:

```
sudo exportfs -a
```

---

Now, You may use the following command at a terminal prompt to restart the NFS server

```
sudo systemctl restart nfs-kernel-server
```

---

then the below command, sudo ufw allow from the dedicated to any port nfs, is used on a Linux system with the firewall tool ufw to configure firewall rules for allowing access to an NFS server. The ip defines the source of the allowed traffic. In this case, it's restricting access to traffic originating from the IP address. You can replace this with a specific IP address or a subnet if you want to allow access from multiple machines.If we use the multiple nodes from CCE clusters as a client then we have to allow those nodes ip which are used to store the data as a `NFS_CLIENT` .

```
sudo ufw allow from 192.143.12.4 to any port nfs
```

---

**Important Consideration:**

Crucial Points to Remember:

Specificity: Although this command permits access from a single IP, if you need to permit access froTo start the NFS server, you can run the following command at a terminal prompt:m several computers, you might want to use a subnet or firewall groups.

Security: It may not always be the best idea to provide access from any port using any. It is advised to specifically identify the ports your NFS server is utilizing if you are aware of them in order to improve security.Crucial Points to Remember:

Configuring the NFS Server: Keep in mind that this operation simply sets up the firewall. Additionally, you must set up the NFS server itself using tools like rpc.mountd or the configuration settings in your NFS server software to permit access from the designated IP address or subnet.

---

**Enable the uncomplicate firewall :**

Once we start the activity of the firewall we've to remember that the SSH connection could be interrupt.So before leaving the virtual machine or local server we need to disable the firewall.

```
sudo ufw enable 
```

--
for disable the firewall

```
sudo ufw disable 
```

---

Now we can check the status of the firewall

```
sudo ufw status 
```

---

Then the next step is i[ntegrating the cce with NFS_server](https://github.com/ahbadhon097/-NFS-Server-Provider-Huawei-Cloud/blob/main/Integrating_NFS_with_CCE.md)
