# Lacework AWS Elastic Beanstalk Walkthrough

## Overview
This is a tutorial for building an AWS Elastic Beanstalk deployment and installing the Lacework Agent. Details will be derived several resources including Github and examples from the web. Elastic Beanstalk is an AWS managed service that’s used for automating the deployment of web applications. AWS handles the creation of the backend infrastructure for hosting the app, so all that you have to provide is the contents of the application. AWS will spin up EC2 instances with application containers used for hosting your application. You provide the docker file and any app directories that contain instructions for initializing and running your application. AWS handles the rest, and allows you to manage your application from both the AWS CLI and the AWS Portal. 

## Requirements
You want to have the following applications installed to follow along with this tutorial:

1. AWS CLI - [Installing or updating the latest version of the AWS CLI - AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  
2. Elastic Beanstalk CLI - [Install the EB CLI - AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)  
3. Docker - [Get Docker](https://docs.docker.com/get-docker/) 
4. Linux OS to run docker and build your app

## Deployment
To build an application to deploy to AWS elastic beanstalk, there are a ton of tutorials on the web, and suggest using the following if this is your first time.

[Deploying a Docker Container to AWS with Elastic Beanstalk](https://sommershurbaji.medium.com/deploying-a-docker-container-to-aws-with-elastic-beanstalk-28adfd6e7e95)

## Deploying a Docker Container to AWS with Elastic Beanstalk  

This guide will walk you through creating an app directory for a simple node.js app, and building a docker container to deploy the app along with an app.js file that contains instructions for launching the app. 

## Access Token
We’ll need an agent access token here, so head to the Lacework UI and create an Agent token

[Create Agent Access Tokens and Download Agent Installers | Lacework Docs](https://docs.lacework.com/create-agent-access-tokens-and-download-agent-installers)
 
To deploy the agent to Elastic beanstalk, we can use what’s called an .ebextension. Ebextensions are plugins for your Elastic Beanstalk application. The agent can run in 3 modes using an .ebextension:

1. As a systemd service on the node
2. As a docker container on the node alongside the app container
3. Within the application container running on the node

The below example installs the agent as a system service on the node whenever a new container is spun up on a node, and calls the latest agent version from the Lacework yum repository:


```files:
  "/var/lib/lacework/config/config.json" :
    mode: "000644"
    owner: root
    group: root
    content: |
      {"tokens": {"Accesstoken": "<AGENT_ACCESS_TOKEN_HERE>"}}
commands:

# Create the agent's yum repository
  "01-lacework-repository":
    command: curl -o /etc/yum.repos.d/lacework-prod.repo https://packages.lacework.net/RPMS/x86_64/lacework-prod.repo
#
# Update your yum cache
  "02-update-yum-cache":
    command: yum -q makecache -y --disablerepo='*' --enablerepo='packages-lacework-prod'
#
# Run the installation script
  "03-run-installation-script":
    command: yum install lacework -y
```

 

Create an ./ebextensions directory within your root app directory that you’ll have created if you followed along with the tutorial provided above:


```node_app$ mkdir .ebextensions
node_app$ chmod 644 .ebextensions
node_app$ cd .ebextensions
.ebextensions$ touch lacework.config


#Using your editor of choice, edit the lacework.config file and paste the above code snippet
#Make sure you place your agent access token in the config and save the file
and either launch your app using the following eb cli commands:


node_app$ eb init
node_app$ eb create
or update your app (if the app is already deployed) using the eb cli command:


node_app$ eb deploy
```

## Caveats
We do run into some known challenges with Elastic Beanstalk as EB uses an Nginx reverse proxy to establish connections to the containers running on the nodes. 

1. You can only use a Network Loadbalancer in Elastic Beanstalk where the "Preserve client IP addresses" is enabled. The Application and Classic Loadbalancer both don't support that.

2. Elastic Beanstalk is using NGINX as a reverse proxy on the EC2 nodes deployed ([Configuring the reverse proxy - AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html)). We only see the traffic from the Network Loadbalancer where the "Preserve client IP addresses" is enabled to the Nginx reverse proxy. From there on we don't see the connection established to the container.

## EB CLI
1. When installing the Elastic Beanstalk CLI, you want to use the preferred python installer as described in the following GitHub page: [GitHub - aws/aws-elastic-beanstalk-cli-setup: Simplified EB CLI installation mechanism](https://github.com/aws/aws-elastic-beanstalk-cli-setup).  
2. On macOS, you can install ebcli using Brew, but you may run into dependency issues as the EB team does not have any control over how Brew operates or is maintained.  


## Other Supported Deployment Methods
1. [As a docker container on the node alongside the app container](https://github.com/automatecloud/lacework-aws-elastic-beanstalk/tree/main/my-tweet-app)
2. [Within the application container running on the node](https://github.com/automatecloud/lacework-aws-elastic-beanstalk/tree/main/my-tweet-app-insidecontainer)
