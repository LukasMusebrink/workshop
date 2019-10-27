# Dockerize

[Working phase 01.03]

## Install Docker
1. Install the most recent Docker Community Edition package\.

   ```bash 
   sudo amazon-linux-extras install docker
   ```

1. Start the Docker service\.

   ```bash
   sudo service docker start
   ```

1. Add the `ec2-user` to the `docker` group so you can execute Docker commands without using `sudo`\.

   ```bash
   sudo usermod -a -G docker ec2-user
   ```

1. Log out and log back in again to pick up the new `docker` group permissions\. You can accomplish this by closing your current SSH terminal window and reconnecting to your instance in a new one\. Your new SSH session will have the appropriate `docker` group permissions\.

1. Verify that the `ec2-user` can run Docker commands without `sudo`\.

   ```bash
   docker info
   ```
**Note**  
In some cases, you may need to reboot your instance to provide permissions for the `ec2-user` to access the Docker daemon\. Try rebooting your instance if you see the following error:  

   ```
   Cannot connect to the Docker daemon. Is the docker daemon running on this host?
   ```
## Dockerize TicketMonster

1. Create a new folder and download all required files

```bash
cd ~
mkdir Dockerize
cd Dockerize
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/dockerize/commands.cli
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/dockerize/customize.sh
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/dockerize/postgresql-9.4-1202.jdbc41.jar
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/dockerize/ticket-monster.war
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/dockerize/Dockerfile
```
**Wait here and let us talk about the files.**

**Now we have to create a RDS Instance together**

```bash
docker build -t ticketmonster .
```

```bash
docker run -d -p 8080:8080 -e DB_HOST=ticketmonster-2.cqqhnwljvdgd.eu-central-1.rds.amazonaws.com -e DB_PORT=5432 -e DB_USERNAME=ticketmonster -e DB_PASSWORD=ticketmonster -e DB_NAME=postgres ticketmonster
```
