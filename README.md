# gcp-vpc

```
gcloud compute instances create nat-gateway --network demo-vpc --can-ip-forward \
    --zone us-central1-a \
    --image-family debian-9 \
    --subnet demo-vpc-public-subnet-us-central \
    --image-project debian-cloud \
    --tags nat-instance
```

```
gcloud compute instances create example-instance --network demo-vpc --no-address \
    --zone us-east1-b \
    --image-family debian-9 \
    --subnet demo-vpc-private-subnet-us-east \
    --image-project debian-cloud \
    --tags private-instance
```

```
gcloud compute routes create demo-vpc-no-ip-internet-route --network demo-vpc \
    --destination-range 0.0.0.0/0 \
    --next-hop-instance nat-gateway \
    --next-hop-instance-zone us-east1-b \
    --tags private-instance --priority 800
```

```
```
