
From a GCP configuration perspective , lets break the config in 2 steps
- Producer specific config
- Consumer specific config
To make things easy for understanding , "producer" in our case is Palo VM exposed behind an internal UDP pass through load balancer . "Consumer" in our case will be VMs sitting in app-1-vpc and app-2-vpc
# Producer specific configuraton
---
### Step 1 :  Configure NSI Load Balancer (iLB2)
Create a second Internal Load Balancer in trust-vpc for NSI 
**Configuration Details:**
* **Type:** Internal Passthrough Network Load Balancer (L4)
* **Protocol:** UDP (GENEVE encapsulation uses UDP port `6081`)
* **Global Access:** Disabled *(must be disabled for NSI)*
* **Frontend VIP:** `192.168.10.8` (example)
* **Backend:** Firewall `nic2` IP (in `trust-vpc`)
* **Health Check:** TCP port `80`
---

### Step 2 : Create Intercept Deployment Group
Create a global Intercept Deployment Group to act as a container for your firewall deployments:
You can do this via GUI or gcloud commands
```bash
gcloud beta network-security intercept-deployment-groups create palo-nsi-deployment-group \
    --location=global \
    --network=projects/ncc-nsi-nw-project/global/networks/trust-vpc \
    --project=ncc-nsi-nw-project
```
---
### Step 3 : Create Zonal Intercept Deployment
Create a zonal deployment in the firewall's zone, pointing it to the forwarding rule of iLB2 (assume forwarding rule is named ilb2-forwarding-rule):
You can do this via GUI or gcloud commands
```bash
gcloud beta network-security intercept-deployments create palo-nsi-deployment-a \
    --location=asia-south1-a \
    --forwarding-rule=projects/ncc-nsi-nw-project/regions/asia-south1/forwardingRules/ilb2-forwarding-rule \
    --intercept-deployment-group=palo-nsi-deployment-group \
    --forwarding-rule-location=asia-south1 \
    --project=ncc-nsi-nw-project
```

After this step , you should see following -->
<img width="1293" height="507" alt="image" src="https://github.com/user-attachments/assets/943f6873-1c7d-4908-8260-7159b9969e20" />
<img width="1293" height="867" alt="image" src="https://github.com/user-attachments/assets/17560ebf-07f6-4561-a976-613aa6ca593c" />

---
# Consumer specific configuration
### Step 4 : prepare Palo Alto firewall
Enable GENEVE Inspection on VM-Series Firewall
To allow the firewall to process GENEVE encapsulated traffic steered by GCP, you must enable GENEVE inspection via the CLI and reboot:

SSH into the VM-Series firewall.
Run the following command to enable inspection
```bash
request plugins vm_series gcp ips inspect enable yes
```
 Reboot the firewall 

---

### Step 5 : Configure App Project 1 (same steps need to be repeated for app2 project also)
Create Intercept Endpoint Group
```bash

gcloud beta network-security intercept-endpoint-groups create app1-nsi-endpoint-group \
    --location=global \
    --intercept-deployment-group=projects/ncc-nsi-nw-project/locations/global/interceptDeploymentGroups/palo-nsi-deployment-group \
    --project=ncc-nsi-app-project
```
Create Intercept Endpoint Group Association: Links app-1-vpc to the endpoint group for inspection.
```bash

gcloud beta network-security intercept-endpoint-group-associations create app1-nsi-association \
    --intercept-endpoint-group=app1-nsi-endpoint-group \
    --location=global \
    --network=projects/ncc-nsi-app-project/global/networks/app-1-vpc \
    --project=ncc-nsi-app-project
```
Create Custom Intercept Security Profile: Links to the endpoint group.
```bash

gcloud network-security security-profiles custom-intercept create app1-nsi-profile \
    --location=global \
    --intercept-endpoint-group=projects/ncc-nsi-app-project/locations/global/interceptEndpointGroups/app1-nsi-endpoint-group \
    --project=ncc-nsi-app-project
```

Create Security Profile Group: Bundles the profile.
```bash
gcloud network-security security-profile-groups create app1-nsi-spg \
    --location=global \
    --custom-intercept-profile=projects/ncc-nsi-app-project/locations/global/securityProfiles/app1-nsi-profile \
    --project=ncc-nsi-app-project
```

Some snapshots from GUI -
### intercept endpoint group
<img width="1427" height="445" alt="image" src="https://github.com/user-attachments/assets/95cf76ff-880c-488f-90db-b7cf969d7c44" />

### association with app VPC
<img width="1760" height="717" alt="image" src="https://github.com/user-attachments/assets/2782b762-2af6-4836-95fd-923c023c63a5" />

### Security profile
<img width="1760" height="504" alt="image" src="https://github.com/user-attachments/assets/7d80ef7d-f31e-4813-992e-76ca96caf500" />

### Security profile group
<img width="1760" height="588" alt="image" src="https://github.com/user-attachments/assets/97e924de-6496-4eac-a906-1ab77239292b" />

---




