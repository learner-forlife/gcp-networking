Step 1: Create VPCs and Subnets
Create the VPC networks and subnets across the 4 projects in asia-south1 region. 

Step 2: Spin up 3P firewall , Generate SSH Key Pair for Firewall Access , login into firewall
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
