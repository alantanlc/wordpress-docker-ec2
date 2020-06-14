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

#### Clone this repository on your instance
```
$ git clone https://gitlab.com/alanwuha/wordpress.git
$ cd wordpress
```

#### Update configuration in `docker-compose.yml` in the wordpress directory
Update [`docker-compose.yml`](./docker-compose.yml) with the credentials (e.g. database name and passwords) and ports specific to your project using an editor before building the project.

If you are unsure about the necessary changes, please contact the writer of this guide for any clarification.

# Building the WordPress project
Run the following command from the wordpress directory.
```
$ sudo docker-compose up -d
```
This runs `sudo docker-compose up` in detached mode, pulls the needed Docker images, and starts the containers.

Containers:
- WordPress [`http://54.255.184.141:80`](http://54.255.184.141:80)
- Prometheus (metrics database) [`http://54.255.184.141:9090`](http://54.255.184.141:9090)
- AlertManager (alerts management) [`http://54.255.184.141:9093`](http://54.255.184.141:9093)
- Grafana (visualize metrics) [`http://54.255.184.141:443`](http://54.255.184.141:443)
- Phpmyadmin [`http://54.255.184.141:82`](http://54.255.184.141:82)
- NodeExporter (host metrics collector) [`http://54.255.184.141:9100/metrics`](http://54.255.184.141:9100/metrics)
- MysqldExporter (database metrics collector) [`http://54.255.184.141:9104/metrics`](http://54.255.184.141:9104/metrics)
- cAdvisor (containers metrics collector) [`http://54.255.184.141:8080/metrics`](http://54.255.184.141:8080/metrics)
- Mysql (database)

# Setting Up WordPress

Navigate to [`http://54.255.184.141:80`](http://54.255.184.141:80) and complete the "famous five-minute installation" as a WordPress administrator.

# Setting Up Grafana

Navigate to [`http://54.255.184.141:443`](http://54.255.184.141:443) and login with user __admin__ password __admin__.

#### Add Prometheus as data source

Go to `Configure > Data Sources` and add Prometheus as a data source. Fill in the URL input with `prometheus:9090` and click on __Save & Test__. If successful, it should return `Data source is working`.

#### Add dashboards

Go to `Create > Import` import the following dashboards one by one using `Import via grafana.com`, select 'Prometheus' under the _Prometheus_ dropdown and the click __Import__:
- 1860 (Node Exporter Full)
- 7362 (MySQL Overview)
- 893 (Docker and system monitoring)

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

#### Instance Down
```
$ sudo docker stop <CONTAINER ID>
```

#### High Storage Load
```
$ fallocate -l 10G dummy.img
```

# Assumptions

This is a standalone application with wordpress, database, and monitoring tools installed on the same AWS EC2 instance.

# References
1. [Best Practices for WordPress on AWS](https://d1.awsstatic.com/whitepapers/wordpress-best-practices-on-aws.pdf)
1. [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
1. [Install Docker Compose on Ubuntu](https://docs.docker.com/compose/install/)
1. [Docker for Beginners: Full Course](https://www.youtube.com/watch?v=zJ6WbK9zFpI)
1. [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)
1. [Docker image: WordPress](https://hub.docker.com/_/wordpress/)
1. [Docker image: MySQL](https://hub.docker.com/_/mysql)
1. [Docker image: PhpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin)
1. [Docker image: Prometheus](https://hub.docker.com/r/prom/prometheus)
1. [Docker image: Grafana](https://hub.docker.com/r/grafana/grafana)
1. [Docker image: Node Exporter](https://hub.docker.com/r/prom/node-exporter/)
1. [Docker image: Mysqld Export](https://hub.docker.com/r/prom/mysqld-exporter)
1. [Docker image: Alertmanager](https://hub.docker.com/r/prom/alertmanager)
1. [Prometheus Docker Installation Guide](https://prometheus.io/docs/prometheus/latest/installation/)
1. [Quick Wordpress Setup with Docker](https://www.youtube.com/watch?v=pYhLEV-sRpY)
1. [Monitoring Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring_ec2.html)
1. [Dockprom - A monitoring solution for Docker hosts and containers](https://github.com/stefanprodan/dockprom)
1. [Setting up Prometheus and Grafana for monitoring your servers](https://www.youtube.com/watch?v=4WWW2ZLEg74)
1. [Step-by-step guide to setting up Prometheus Alertmanager with Slack, PagerDuty, and Gmail](https://grafana.com/blog/2020/02/25/step-by-step-guide-to-setting-up-prometheus-alertmanager-with-slack-pagerduty-and-gmail/)
1. [Dockprom alert rules](https://github.com/stefanprodan/dockprom/blob/master/prometheus/alert.rules)
1. [Save your Grafana data to Docker volume](https://grafana.com/docs/grafana/latest/installation/configure-docker/#save-your-grafana-data)
1. [Docker Data Volume Container pattern](https://docs.docker.com/storage/volumes/)