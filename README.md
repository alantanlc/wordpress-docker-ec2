# Installing WordPress on Amazon Elastic Compute Cloud (Amazon EC2) Ubuntu 18.04 using Docker Compose

There are three main ways to host your own WordPress website on AWS according to the [best practices](https://d1.awsstatic.com/whitepapers/wordpress-best-practices-on-aws.pdf):

1. Amazon Lightsail (Recommended)
1. Amazon Elastic Compute Cloud (Amazon EC2)
1. AWS Marketplace

There are also different ways to install WordPress on a Ubuntu machine (non-exhaustive):
1. Manually set up WordPress with LAMP stack like [this](https://linuxbeast.com/tutorials/aws/how-to-install-wordpress-on-ec2-ubuntu-18-04/) or [this](https://medium.com/@kavishbaghel/setting-up-wordpress-with-lamp-stack-on-ec2-instance-aws-articles-28e50a675e08) or [this](https://aws.amazon.com/getting-started/hands-on/deploy-wordpress-with-amazon-rds/)
1. Using containers (e.g. Docker and Docker Compose) like [this](https://docs.docker.com/compose/wordpress/)
1. Using infrastructure as code tools (e.g. Terraform, Ansible) like [this](https://medium.com/@mschirbel/wordpress-on-aws-using-terraform-and-ansible-8c3e04cb76e9)

This guide covers the steps to host a __WordPress on a Amazon Elastic Compute Cloud (Amazon EC2) Ubuntu 18.04 machine using Docker Compose__. You can use Docker Compose to easily run WordPress in an isolated environment built with Docker containers.

# Connecting to the AWS EC2 instance

#### Prepare private key on your local machine

The [private key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) (a `.pem` file) that corresponds to your AWS EC2 instance is required in order to connect to your instance using SSH to perform the WordPress installation. If you do not have the private key, please check with your instance provider.

Extract the private key from the zipped file.
Note that it is not '-x', there is no space between the option '-p' and the password for the `7z` command. Replace `PASSWORD` with the password for the zipped file.
```
$ sudo apt-get install p7zip-full
$ 7z x -pPASSWORD devops-test.pvt.zip
$ mv devops-test.pvt.key devops-test.pem
$ chmod 400 devops-test.pem
```

#### Connect to your AWS EC2 instance using SSH
```
$ ssh -i devops-test.pem devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com
```

# Installing Docker Engine and Docker Compose

If the instance already has Docker Engine and Docker Compose installed, you can skip to [Setting up the WordPress project](#setup).

#### Update software repositories and packages
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

#### Install Docker Engine
This sets up the Docker repository and installs the _latest version_ of Docker Engine and containerd.
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Verify that Docker Engine is installed correctly  by running the `hello-world` image.
```
$ sudo docker run hello-world
```

For more information on installing, uninstalling and upgrading Docker Engine, check out the [official installation guide](https://docs.docker.com/engine/install/ubuntu/).

#### Install Docker Compose
This downloads the _current stable release_ of Docker Compose binary from the [Compose repository release page on GitHub](https://github.com/docker/compose/releases).
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Test the installation.
```
$ sudo docker-compose --version
docker-compose version 1.26.0, build 1110ad01
```

For more information on installing, uninstalling and upgrading Docker Compose, check out the [official installation guide](https://docs.docker.com/compose/install/).

<a name='setup'></a>

# Setting up the WordPress project

#### Clone the wordpress repository hosted on GitLab to your instance
```
$ git clone https://gitlab.com/alanwuha/wordpress.git
$ cd wordpress
```

#### Update configuration in `docker-compose.yml` in the wordpress directory
Update `docker-compose.yml` with the credentials (e.g. database name and passwords) and ports specific to your project using an editor before building the project.

If you are unsure about the necessary changes, please contact the writer of this guide for any clarification.

# Building the WordPress project
Run the following command from the wordpress directory.
```
$ sudo docker-compose up -d
```
This runs `sudo docker-compose up` in detached mode, pulls the needed Docker images, and starts the wordpress and database containers.

#### Bring up WordPress in a web browser

At this point, WordPress should be running on port `80` of your Docker Host, and you can complete the "famous five-minute installation" as a WordPress administrator by accessing the public IP address of your instance on port `80` (e.g. http://54.255.184.141:80) on the web browser.

#### Bring up PhpMyAdmin in a web browser

At this point, PhpMyAdmin should be running on port `8080` of your Docker Host, and you can manage your database by accessing the public IP address of your instance on port `8080` (e.g. http://54.255.184.141:8080) on the web browser.

#### Bringing up Prometheus in a web browser

At this point, Prometheus should be running on port `9090` of your Docker Host, and you can perform queries and set up alerts or graphs by accessing the public IP address of your instance on port `9090` (e.g. http://54.255.184.141:9090) on the web browser.

#### Bringing up Grafana in a web browser

At this point, Grafana should be running on port `443` of your Docker Host, and you can add data sources and create dashboards by accessing the public IP address of your instance on port `443` (e.g. http://54.255.184.141:443) on the web browser.

# Shutting down and cleaning up

Remove the containers and default network, but preserve your WordPress database.
```
$ sudo docker-compose down
```

Remove the containers, default network, and the WordPress database. __DO THIS WITH CAUTION AS YOUR DATA WILL BE ERASED.__
```
$ sudo docker-compose down --volumes
```

# Monitoring Test

#### Filesystem space available
```
fallocate -l 18G gentoo_root.img
```

# Assumptions

This is a standalone application with wordpress, database, and monitoring tools installed on the same AWS EC2 instance.

# Architecture Overview

insert image here


# References
1. [Best Practices for WordPress on AWS](https://d1.awsstatic.com/whitepapers/wordpress-best-practices-on-aws.pdf)
1. [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)
1. [Docker Image: WordPress](https://hub.docker.com/_/wordpress/)
1. [Docker image: MySQL](https://hub.docker.com/_/mysql)
1. [Docker image: PhpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin)
1. [Docker image: Prometheus](https://hub.docker.com/r/prom/prometheus)
1. [Docker image: Grafana](https://hub.docker.com/r/grafana/grafana)
1. [Docker image: Node Exporter](https://hub.docker.com/r/prom/node-exporter/)
1. [Prometheus Docker Installation Guide](https://prometheus.io/docs/prometheus/latest/installation/)
1. [Quick Wordpress Setup with Docker](https://www.youtube.com/watch?v=pYhLEV-sRpY)
1. [Dockprom - A monitoring solution for Docker hosts and containers](https://github.com/stefanprodan/dockprom)
1. [Setting up Prometheus and Grafana for monitoring your servers](https://www.youtube.com/watch?v=4WWW2ZLEg74)
1. [Docker Data Volume Container pattern](https://docs.docker.com/storage/volumes/)
1. [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
1. [Install Docker Compose on Ubuntu](https://docs.docker.com/compose/install/)
1. [Monitoring Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring_ec2.html)
1. [Docker for Beginners: Full Course](https://www.youtube.com/watch?v=zJ6WbK9zFpI)
1. [Save your Grafana data](https://grafana.com/docs/grafana/latest/installation/configure-docker/#save-your-grafana-data)