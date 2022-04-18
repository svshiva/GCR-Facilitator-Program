## Task 1: Create a project jumphost instance

Navigation menu > Compute engine > VM Instance

Create Instance with following values

-   Name: nucleus-jumphost (Or as defined in Lab Manual)
-   Machine Type: **f1-micro**
-   Use the default image type (Debian Linux).



## Task 2: Create a Kubernetes service cluster
```
gcloud config set compute/zone us-east1-b

gcloud container clusters create nucleus-webserver1

gcloud container clusters get-credentials nucleus-webserver1

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-app --type=LoadBalancer --port 8080

kubectl get service 
```

## Task 3: Setup an HTTP load balancer

### Open the cloud shell and Copy and Paste the following commands
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```


1.   Create an instance template :
```
gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh

```

2.   Create a target pool :
```
gcloud compute target-pools create nginx-pool
```

3.   Create a managed instance group :

```
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool

gcloud compute instances list
```

4. Create a firewall rule to allow traffic (80/tcp) :

```
gcloud compute firewall-rules create www-firewall --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool

gcloud compute forwarding-rules list
```

5. Create a health check :

```
gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80
```

6. Create a backend service and attach the manged instance group :

```
gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global
```

7. Create a URL map and target HTTP proxy to route requests to your URL map :

```
gcloud compute url-maps create web-map \
--default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map
```

8. Create a forwarding rule :

```
gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80
```

9. Check you rule

```
gcloud compute forwarding-rules list
```

>You may need to wait for 5-10 mins after completing last step.

Now Check your progress

### **Congratulations! You completed this challenge lab.**

View Solution to More Challenge Labs:

[Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab](track1/PerformFoundationalInfrastructureTasksInGoogleCloud.md)


[Build and Secure Networks in Google Cloud: Challenge Lab](track1/BuildAndSecureNetworksInGoogleCloud.md)

[Deploy to Kubernetes in Google Cloud: Challenge Lab](track1/DeployToKubernetesInGoogleCloud.md)

[Setup and Configure a Cloud Environment in Google Cloud: Challenge Lab](track1/SetupAndConfigureACloudEnvironmentInGoogleCloud.md)

[Implement DevOps in Google Cloud: Challenge Lab](track1/ImplementDevOpsInGoogleCloud.md)

[Google Cloud Essential Skills: Challenge Lab](track1/GoogleCloudEssentialSkills.md)

[Monitor and Log with Google Cloud Operations Suite: Challenge Lab](track1/MonitorAndLogWithGoogleCloudOperationsSuite.md)