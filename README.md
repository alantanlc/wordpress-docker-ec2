# Installing WordPress on Amazon Elastic Compute Cloud (Amazon EC2) Ubuntu 18.04 using Docker Compose

There are three main ways to host your own WordPress website on AWS:

1. Amazon Lightsail (Recommended)
1. Amazon Elastic Compute Cloud (Amazon EC2)
1. AWS Marketplace

There are also different ways to install WordPress on a Ubuntu machine (non-exhaustive):
1. Manually set up WordPress with LAMP stack like [this](https://linuxbeast.com/tutorials/aws/how-to-install-wordpress-on-ec2-ubuntu-18-04/) or [this](https://medium.com/@kavishbaghel/setting-up-wordpress-with-lamp-stack-on-ec2-instance-aws-articles-28e50a675e08) or [this](https://aws.amazon.com/getting-started/hands-on/deploy-wordpress-with-amazon-rds/)
1. Using containers (e.g. Docker and Docker Compose) like [this](https://docs.docker.com/compose/wordpress/)
1. Using infrastructure as code tools (e.g. Terraform, Ansible) like [this](https://medium.com/@mschirbel/wordpress-on-aws-using-terraform-and-ansible-8c3e04cb76e9)

This guide covers the steps to host a WordPress on a Amazon Elastic Compute Cloud (Amazon EC2) Ubuntu 18.04 machine using Docker Compose. You can use Docker Compose to easily run WordPress in an isolated environment built with Docker containers.

# Logging into the AWS EC2 instance

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

#### Connect into your AWS EC2 instance using SSH
```
$ ssh -i devops-test.pem devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com
```

# Installing WordPress on the AWS EC2 instance

#### Update and install system packages
```
$ sudo apt update && sudo apt upgrade
```

#### Install Docker and Docker Compose
```
$ sudo apt install docker.io
$ sudo apt install docker-compose
```

#### Create a wordpress project using git or wget
Using git:
```
$ git clone https://github.com/alanwuha/wordpress.git
$ cd wordpress
```

Using wget:
```
$ mkdir wordpress
$ cd wordpress
$ wget https://raw.githubusercontent.com/alanwuha/edb/master/docker-compose.yml
```

#### Update configuration in `docker-compose.yml`
Update `docker-compose.yml` with the credentials (e.g. database name and passwords) and ports specific to your project using an editor before building the project.

If you are unsure about the necessary changes, please contact the writer of this guide for any clarification.

#### Build the project
Run the following command from the wordpress directory.
```
$ sudo docker-compose up -d
```
This runs `sudo docker-compose up` in detached mode, pulls the needed Docker images, and starts the wordpress and database containers.

#### Bring up WordPress in a web browser

At this point, WordPress should be running on port `80` of your Docker Host, and you can complete the "famous five-minute installation" as a WordPress administrator by accessing the public IP address of your instance on port `80` (e.g. http://54.255.184.141:80) on the web browser.

#### Bring up PhpMyAdmin in a web browser

At this point, PhpMyAdmin should be running on port `443` of your Docker Host, and you can manage your database by accessing the public IP address of your instance on port `443` (e.g. http://54.255.184.141:443) on the web browser.

# Shutting down and cleaning up

Remove the containers and default network, but preserve your WordPress database.
```
$ sudo docker-compose down
```

Remove the containers, default network, and the WordPress database. __DO THIS WITH CAUTION, YOUR DATA WILL BE ERASED AND THERE IS NO GOING BACK.__
```
$ sudo docker-compose down --volumes
```

# References
1. [Best Practices for WordPress on AWS](https://d1.awsstatic.com/whitepapers/wordpress-best-practices-on-aws.pdf)
1. [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)
1. [Docker image: WordPress](https://hub.docker.com/_/wordpress/)
1. [Docker image: MySQL](https://hub.docker.com/_/mysql)
1. [Docker image: PhpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin)
1. [Quick Wordpress Setup with Docker](https://www.youtube.com/watch?v=pYhLEV-sRpY)