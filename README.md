# Installing WordPress on Amazon Elastic Compute Cloud (Amazon EC2) using Docker Compose

There are three main ways to host your own WordPress website on AWS:

1. Amazon Lightsail
1. Amazon Elastic Compute Cloud (Amazon EC2)
1. AWS Marketplace

This document covers the steps to host a WordPress on a Amazon Elastic Compute Cloud (Amazon EC2) using Docker Compose.

You can use Docker Compose to easily run WordPress in an isolated environment built with Docker containers. This quick start guide demonstrates how to use Docker Compose to set up and run WordPress on a AWS EC2 instance.

# Logging into the AWS EC2 instance

### Prepare Private Key

The [private key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) (a `.pem` file) that corresponds to your AWS EC2 instance is required in order to connect to your instance using SSH to perform the WordPress installation. If you do not have the private key, please check with your instance provider.

Extract the private key from the zipped file.
Note that it is not '-x', there is no space between the option '-p' and the password for the `7z` command. Replace `PASSWORD` with the password for the zipped file.
```
$ sudo apt-get install p7zip-full
$ 7z x -pPASSWORD devops-test.pvt.zip
$ mv devops-test.pvt.key devops-test.pem
$ chmod 400 devops-test.pem
```

### SSH into your AWS EC2 instance
```
$ ssh -i devops-test.pem devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com
```

# Installing WordPress on the AWS EC2 instance

### Update and install system packages
```
$ sudo apt update && sudo apt upgrade
```

### Install Docker and Docker Compose
```
$ sudo apt install docker.io
$ sudo apt install docker-compose
```

### Create a wordpress directory
```
$ mkdir wordpress
$ cd wordpress
```

### Copy `docker-compose.yml` into the wordpress directory using wget or scp
Using wget:
```
$ wget https://gist.githubusercontent.com/alanwuha/820f7d11c44abcb13155867ed54c431e/raw/9ac2fb951df88466afe71d8ca474f0fe1d1d746d/docker-compose.yml
```

Using scp:
```
$ scp -i devops-test.pvt.key -r docker-compose.yml devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com:~/wordpress
```

### Update `docker-compose.yml`
Be sure to update `docker-compose.yml` with the credentials and ports specified to your project before building the project.

### Build the project
Run the following command from the wordpress directory.
```
$ sudo docker-compose up -d
```
This runs `sudo docker-compose up` in detached mode, pulls the needed Docker images, and starts the wordpress and database containers.

### Bring up WordPress in a web browser

At this point, WordPress should be running on port `80` of your Docker Host, and you can complete the "famous five-minute installation" as a WordPress administrator by accessing the public IP address of your instance on port `80` (e.g. http://54.255.184.141:80) on the web browser.

### Bring up PhpMyAdmin in a web browser

At this point, PhpMyAdmin should be running on port `443` of your Docker Host, and you can manage your database by accessing the public IP address of your instance on port `443` (e.g. http://54.255.184.141:443) on the web browser.

# Shutting down and cleaning up

The command `docker-compose down` removes the containers and default network, but preserves your WordPress database.

The command `docker-compose down --volumes` removes the containers, default network, and the WordPress database.

# References
1. [Best Practices for WordPress on AWS](https://d1.awsstatic.com/whitepapers/wordpress-best-practices-on-aws.pdf)
1. [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)
1. [Quick Wordpress Setup with Docker](https://www.youtube.com/watch?v=pYhLEV-sRpY)