# gcp-vpc

## create vpc

```
gcloud compute networks create demo-vpc \
    --subnet-mode=custom \
    --bgp-routing-mode=regional \
    --mtu=1460
```

## add subnet
```
gcloud compute networks subnets create demo-vpc-public-subnet-us-central \
    --network=demo-vpc \
    --range=10.0.1.0/24 \
    --region=us-central1
```
```
gcloud compute networks subnets create demo-vpc-private-subnet-us-east \
    --network=demo-vpc \
    --range=10.0.2.0/24 \
    --region=us-east1
```

1. Create 1st firewall rule to allow SSH access .

```
gcloud compute firewall-rules create demo-vpc-allow-ssh --allow tcp:22 --network demo-vpc
```
2. Create 2nd firewall rule to allow access (tcp,udp,icmp ports)internally within VPC
```
gcloud compute firewall-rules create demo-vpc-allow-internal-network --allow tcp:1-65535,udp:1-65535,icmp --source-ranges 10.0.0.0/16 --network demo-vpc
```
3. Launch NAT instance in Public subnet with tag as “nat-instance”.
```
gcloud compute instances create nat-gateway --network demo-vpc --can-ip-forward \
    --zone us-central1-a \
    --image-family debian-9 \
    --subnet demo-vpc-public-subnet-us-central \
    --image-project debian-cloud \
    --tags nat-instance
```

note: follow this link to [create cloud nat](https://cloud.google.com/nat/docs/using-nat#creating_nat)
add a nat in the region where private subnet resides

4. Launch 2 instances in private subnet with tag as “Private-instances”
```
gcloud compute instances create example-instance --network demo-vpc --no-address \
    --zone us-east1-b \
    --image-family debian-9 \
    --subnet demo-vpc-private-subnet-us-east \
    --image-project debian-cloud \
    --tags private-instance
```
5. Create a route to allow all instances running in private subnet to access internet.
```
gcloud compute routes create demo-vpc-no-ip-internet-route --network demo-vpc \
    --destination-range 0.0.0.0/0 \
    --next-hop-instance nat-gateway \
    --next-hop-instance-zone us-east1-b \
    --tags private-instance --priority 800
```
SSH to your private instances and install apache2 package.
```
sudo apt-get update -y
sudo apt-get install apache2 -y
```
To start Apache 2 web server, enter:
`sudo /etc/init.d/apache2 start`

allow http:
`gcloud compute firewall-rules create demo-vpc-allow-http --allow tcp:80 --network demo-vpc`

refer to this stack overflows [link](https://stackoverflow.com/questions/48515928/how-to-create-private-and-public-subnets-in-gcp)

[example to deploy nodejs in gcp vm](https://medium.com/the-node-js-collection/time-to-hello-world-part2-gce-95e9df907b93)

[How to Check Remote Ports are Reachable Using ‘nc’ Command](https://www.tecmint.com/check-remote-port-in-linux/)
[Deploying the Application — Google Cloud Platform (Public/Private Subnets)](https://medium.com/@manoj.bhagwat60/deploying-the-application-google-cloud-platform-public-private-subnets-9b77986ebd51)
