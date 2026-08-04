# Topology 
<img width="1557" height="707" alt="image" src="https://github.com/user-attachments/assets/4086b964-2ffc-452a-9f7f-b0dcce3a5b2f" />


# Pre Requisites
- Customer already has a working GCP environment as documented in repo "ncc-plus-3pfw" . Meaning thereby
    a) NCC setup already in place . 3 VPCs are attached to NCC hub -> Trust VPC , app-1-vpc , app-2-vpc
    b) GCP to on prem traffic is getting inspected via 3P Palo Alto firewall in GCP . GCP to on-prem communication is working fine as expected

# New Requirements
- No new set of Palo Alto firewall to be used
- Dont change the NCC topology ( which is currently set to mesh )
- app-1-vpc workloads should get inspected at Palo Alto before communicating to app-2-vpc workload

# Why this design needs NSI 
- When we connect 2 VPCs using NCC , their respective subnet routes are auto programmed . Hence , any communication between workloads in these 2 VPC will directly happen and firewall will not come in play
- Example : routing table of app-2 VPC looks like following
  <img width="1746" height="725" alt="image" src="https://github.com/user-attachments/assets/533629aa-7e89-4799-80b8-eaaab7d56c9e" />

  



