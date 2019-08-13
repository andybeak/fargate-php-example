# Fargate PHP example

This simple project shows how to dockerise PHP and Nginx and run them together in AWS Fargate.

## Set up

The high-level steps to get this running are:

1. Get AWS CLI installed and configured so you can push containers to ECR
1. Build the docker containers and push them to ECR
1. Create a Fargate task that references the containers in ECR
1. Create a Fargate cluster
1. Add a load balancer
1. Add a service to the cluster that will run the task
1. Link the load-balancer to the service
1. Add TLS encryption to the load-balancer

### Create ECR repositories
Create one for php-info and one for webserver.

To get the push commands you can click onto the details of the repository and then click the button on the top right that says "View push commands"

Build the docker images and push them to your repositories:

    docker build ./php-info/ -t phpinfo:latest
    docker build ./webserver/ -t webserver:latest
    docker tag php-info:latest <your repo prefix>.amazonaws.com/php-info:latest
    docker tag webserver:latest <your repo prefix>.amazonaws.com/webserver:latest
    docker push <your repo prefix>.amazonaws.com/webserver:latest
    docker push <your repo prefix>.amazonaws.com/webserver:latest

### Create a Fargate task

Navigate to the ECS service and choose task definitions then "create new task definition".

Choose "Fargate"

Task definition name:  php-info
Task role: None
Network mode: This can't be changed because you've chosen Fargate deployment
Task Memory: 1GB
Task vCPU: 0.5

Now add a container.  Name it "php-info" and copy in the repository url for the php-info container.  Leave the rest blank and click add.

Add another container.  Name it "webserver" and use the repository url for the webserver container.
Add a host port mapping and select host port 80 to container port 80.

Finish creating the task.

### Create a cluster to run the task on

Select Clusters in the ECS service menu on the left and add a cluster.

Select "Fargate" deployment (networking only)

Name the cluster php-example-cluster

Leave the rest of the fields as is and click create

### Add a security group for the load balancer

Navigate to the EC2 service and select security groups.  Create a new security group.

Security group name: public web

Description: Allow public access to common web ports

Inbound rules:

HTTP: allow

HTTPS: allow

### Add a load balancer

Navigate to the EC2 service and select Load balancers.  Create a new load balancer and name it "php-example-lb".

The scheme must be internet facing and select the public subnets you want it in.

Click next, ignore the warning about HTTPS (you can add HTTPS later)

Click next for security groups.

Select the security group you added earlier

Click next to configure routing

Add a new target group called "fargate-target"

Click next to register targets

None will be available so skip this step.

Review and create

### Create Fargate service

Navigate to the ECS service, select clusters and click on the cluster you created.

Select "Create" under the services tab

The launch type must be "Fargate"

Name the service fargate-service-example

Set the number of tasks to 1

Click next step

Select the VPC and public subnets (should match the load-balancer I think)

Auto-assign public IP: enabled

Set the load balancer option to "application load balancer"

In the drop down change the load balancer name to "php-example-lb"

Add the container to the load-balancer (select the webserver:80:80 in the drop down)

Set the production listener port to 80

Set the path pattern to /* and set the target group name to "fargate-target"

Leave the rest as is and click to go to the next step

Select "Do not adjust the services desired count" and click next.

Create the service and wait for it to spin up.

### Change the instance health check
Navigate to the EC2 service and select the load-balancer.

Select the target group.

Select health checks and change the path to "/healthcheck.html".

At this point your load balancer should start picking up the healthy host and you can navigate to the load balancer public IP.

### Viewing our PHP page

If you visit http://<your loadbalancer public dns>/index.php you should see a phpinfo page showing PHP 7.3.8