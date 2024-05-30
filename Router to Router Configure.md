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
From a device in the 192.168.1.0/24 subnet, ping a device in the 10.0.0.0/24 subnet to verify connectivity.
Similarly, from a device in the 10.0.0.0/24 subnet, ping a device in the 192.168.1.0/24 subnet.
Additional Configuration (if needed)
Firewall Rules:

Ensure that the firewall rules allow IPSec traffic (UDP port 500, ESP protocol).
Go to IP > Firewall > Filter Rules and NAT Rules to adjust any rules that might be blocking IPSec traffic.
