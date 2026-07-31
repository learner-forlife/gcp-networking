## Phase 1 BASE setup##

**Step 1**: Create VPCs and Subnets
Create the VPC networks and subnets across the 4 projects in asia-south1 region. 

**Step 2**: Spin up 3P firewall , Generate SSH Key Pair for Firewall Access , login into firewall
Generate an SSH key pair in Cloud Shell to authorize the admin user on the Palo Alto firewall. (Verified: Key pair generated and formatted correctly). 

I generally forget commands to generate key pair and use them in market place deployment , hence putting commands here 
Generate the key pair:
bash

ssh-keygen -t rsa -b 2048 -f ~/.ssh/paloalto-key -C admin
(Press Enter to skip passphrase when prompted)
Display the formatted public key:

echo "admin:$(cat ~/.ssh/paloalto-key.pub)"
Copy the output: Copy the entire printed line (which starts with admin:ssh-rsa ...).

This is how your NIC in 3P firewall may look like
<img width="1449" height="413" alt="image" src="https://github.com/user-attachments/assets/f683b09d-d5ab-46be-ad05-824ff7b41469" />

**Step 3**: Establish On-Premises VPN Connectivity
Configure Cloud VPN in connectivity-vpc to connect to onprem-vpc (192.168.100.0/24).
Ensure routes are established in connectivity-vpc to reach On-Premises.
from connectivity VPC --> ensure that you are advertising custom routes 10.10.10.0/24 and 10.20.20.0/24 towards on prem over the IPSEC tunnels
Check that these 2 CIDR are learnt using BGP and NH is IPSEC tunnels

## Phase 2: Firewall Deployment & Initial Setup##

**Step 4**
Deploy a Palo Alto VM-Series firewall instance from GCP Marketplace in Project using the GCP Console GUI.
License: VM-Series Next-Generation Firewall Bundle 2 (PAYG).
VM Type: n2-standard-4 (required for 4 network interfaces).
NIC Mapping (with Management Swap):
nic0: untrust-vpc (IP: 192.168.20.5) 
nic1: mgmt-vpc (IP: 192.168.40.5) - Management (Swapped)
nic2: trust-vpc (IP: 192.168.10.5) 
nic3: connectivity-vpc (IP: 192.168.30.5) 

**Step5**
GUI Settings:
Enable "Swap Management Interface" checkbox.
Enable "Enable Serial Console Access" checkbox.
Paste the SSH key from Step 2 (format: admin:ssh-rsa ...).
Enable "IP Forwarding".

**Step6**
Initial Password Setup:
SSH to management IP (e.g. 34.x.x.x public IP ) using private key: ssh -i ~/.ssh/paloalto-key admin@[MGMT_IP]
Run CLI commands to set password for Web UI access:

configure
set mgt-config users admin password
Enter password (must be >= 8 chars, 1 upper, 1 lower, 1 number/special)
commit
exit

**Step7**
Once you login to Palo Alto 
  - configure interfaces
  - configure VRs
  - configure security zones

in my example , I have 3 VRs ( trust , untrust , Connectivity ) and 3 sec zones as follows

<img width="868" height="220" alt="image" src="https://github.com/user-attachments/assets/d4718970-7253-4068-ba3b-a636960fe346" />
<img width="868" height="220" alt="image" src="https://github.com/user-attachments/assets/f612812c-605f-4fff-a8e3-77f2608dbe0e" />


<img width="729" height="173" alt="image" src="https://github.com/user-attachments/assets/2c2894ed-c870-4633-9296-b643b485cc17" />
<img width="472" height="282" alt="image" src="https://github.com/user-attachments/assets/99842d86-1602-4c93-b3ad-e735207c3b1a" />

## Phase3 : Health Checks
Because we are using L4 pass through iLB , we need to ensure the health check pass.
Below steps are for ILB01 in Trust VPC , same steps can be repeated for other ILB as well
a) ILB address 192.168.10.6 . So we need to create the loopback address on palo Alto with this address and put it in correct VR
<img width="1012" height="358" alt="image" src="https://github.com/user-attachments/assets/f370ebba-42dc-4a95-aa3b-85187d7204a2" />
<img width="670" height="384" alt="image" src="https://github.com/user-attachments/assets/738aa054-6f3f-48e2-8b56-87dbd7804a5f" />

now we need static routes for health check range 
- the below static route is added in VR-Trust
<img width="1162" height="692" alt="image" src="https://github.com/user-attachments/assets/8cb7105e-cf8c-4067-9214-854373a17e70" />

## Phase 4 : NCC related configurations






