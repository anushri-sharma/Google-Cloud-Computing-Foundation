# GSP313

# [Implement Load Balancing on Compute Engine: Challenge Lab](https://www.skills.google/paths/36/course_templates/648/labs/613025)

[check](https://www.youtube.com/watch?v=yopNHlP8qqM)

## Checkpoints
- Create multiple web server instances
- Configure the load balancing service
- Create an HTTP load balancer


# Overview
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!


# Create web1 VM with Apache installed
gcloud compute instances create web1 \
  --zone=europe-west1-b \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=network-lb-tag \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'

# Create web2 VM with Apache installed
gcloud compute instances create web2 \
  --zone=europe-west1-b \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=network-lb-tag \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'

# Create web3 VM with Apache installed
gcloud compute instances create web3 \
  --zone=europe-west1-b \
  --machine-type=e2-small \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=network-lb-tag \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'

# Create firewall rule to allow HTTP traffic
gcloud compute firewall-rules create www-firewall-network-lb \
  --network=default \
  --allow=tcp:80 \
  --target-tags=network-lb-tag \
  --source-ranges=0.0.0.0/0

# Create HTTP health check for target pool
gcloud compute http-health-checks create network-lb-health-check

# Create target pool and attach health check
gcloud compute target-pools create www-pool \
  --region=europe-west1 \
  --http-health-check=network-lb-health-check

# Add instances to the target pool
gcloud compute target-pools add-instances www-pool \
  --instances=web1,web2,web3 \
  --instances-zone=europe-west1-b

# Reserve a static IP for the network load balancer
gcloud compute addresses create network-lb-ip-1 \
  --region=europe-west1

# Create forwarding rule to direct traffic to target pool
gcloud compute forwarding-rules create www-rule \
  --region=europe-west1 \
  --ports=80 \
  --address=network-lb-ip-1 \
  --target-pool=www-pool

# Create instance template for backend servers
gcloud compute instance-templates create lb-backend-template \
   --region=europe-west1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-12 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'

# Create managed instance group using the template
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=europe-west1-b

# Create firewall rule to allow health checks from Google
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80

# Reserve a global IPv4 address for the HTTP load balancer
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global

# Display the global IP address
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global

# Create HTTP health check for backend service
gcloud compute health-checks create http http-basic-check \
  --port 80

# Create backend service and attach health check
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

# Add instance group to backend service
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=europe-west1-b \
  --global

# Create URL map to route traffic to backend service
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

# Create target HTTP proxy linked to URL map
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

# Create forwarding rule for global HTTP load balancer
gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1 \
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80
