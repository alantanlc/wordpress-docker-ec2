# WordPress Installation Guide

# Extract private key 

Extract private key from zipped file.
Note that it is not '-x' and there is no space between the option '-p' and the password.
```
$ sudo apt-get install p7zip-full
$ 7z x -p<PASSWORD> devops-test.pvt.zip
```

Set file permissions of private key file to 600.
This enables read and write permissions to `user` and disables read, write and execute permissions to `group` and `other`. 
```
$ chmod 400 devops-test.pvt.key
```

# SSH into EC2 instance
```
$ ssh -i devops-test.pvt.key devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com
```

# Update packages 
```
$ sudo apt update && sudo apt upgrade
```

# Install docker
```
$ sudo apt install docker.io
```

# Install docker compose
```
$ sudo apt install docker-compose
```

# Create a wordpress directory
```
$ mkdir wordpress
$ cd wordpress
```

# From your local machine, copy docker-compose.yml file into wordpress directory on the EC2 instance
```
$ scp -i devops-test.pvt.key -r docker-compose.yml devops-test@ec2-54-255-184-141.ap-southeast-1.compute.amazonaws.com:~/wordpress
```

# Build the project
Run `sudo docker-compose up -d` from the wordpress directory.

This runs `sudo docker-compose up` in detached mode, pulls the needed Docker images, and starts the wordpress and database containers.

# Bring up WordPress in a web browser

At this point, WordPress should be running on port `80` of your Docker Host, and you can complete the "famous five-minute installation" as a WordPress administrator.

# Shutdown and cleanup

The command `docker-compose down` removes the containers and default network, but preserves your WordPress database.

The command `docker-compose down --volumes` removes the containers, default network, and the WordPress database.
