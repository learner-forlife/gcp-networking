
From a GCP configuration perspective , lets break the config in 2 steps
- Producer specific config
- Consumer specific config
To make things easy for understanding , "producer" in our case is Palo VM exposed behind an internal UDP pass through load balancer . "Consumer" in our case will be VMs sitting in app-1-vpc and app-2-vpc

# Producer specific configuraton
**Step 1** :  Configure NSI Load Balancer (iLB2)
Create a second Internal Load Balancer in trust-vpc for NSI 
Load Balancer Type: Internal Passthrough Network Load Balancer (L4).
Protocol: UDP (GENEVE encapsulation uses UDP port 6081).
Global Access: Disabled (must be disabled for NSI).
Frontend VIP: 192.168.10.8 (example)
Backend: Firewall nic2 IP (in trust Vpc).
Health Check: TCP port 80 

**Step 2** : Create Intercept Deployment Group
Create a global Intercept Deployment Group to act as a container for your firewall deployments:
You can do this via GUI or gcloud commands
`
gcloud beta network-security intercept-deployment-groups create palo-nsi-deployment-group \
    --location=global \
    --network=projects/ncc-nsi-nw-project/global/networks/trust-vpc \
    --project=ncc-nsi-nw-project
`
**Step 3**: Create Zonal Intercept Deployment
Create a zonal deployment in the firewall's zone, pointing it to the forwarding rule of iLB2 (assume forwarding rule is named ilb2-forwarding-rule):
You can do this via GUI or gcloud commands
`
gcloud beta network-security intercept-deployments create palo-nsi-deployment-a \
    --location=asia-south1-a \
    --forwarding-rule=projects/ncc-nsi-nw-project/regions/asia-south1/forwardingRules/ilb2-forwarding-rule \
    --intercept-deployment-group=palo-nsi-deployment-group \
    --forwarding-rule-location=asia-south1 \
    --project=ncc-nsi-nw-project
`

After this step , you should see following -->
<img width="1293" height="507" alt="image" src="https://github.com/user-attachments/assets/943f6873-1c7d-4908-8260-7159b9969e20" />
<img width="1293" height="867" alt="image" src="https://github.com/user-attachments/assets/17560ebf-07f6-4561-a976-613aa6ca593c" />



