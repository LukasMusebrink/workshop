# TicketMonster - a demo application
[Working phase 01.02]

TicketMonster is an online ticketing demo application that gets you started with JBoss technologies, and helps you learn and evaluate them.
Here are a few instructions for building and running it. 
<details>
  <summary>Not required Building TicketMonster</summary>
**Building TicketMonster**
TicketMonster can be built from Maven, by runnning the following Maven command:

```bash
mvn clean package
```
</details>


## Prepare EC2 Instance

```bash
sudo yum update -y
sudo yum upgrade -y
sudo yum autoremove -y
sudo yum install java-11-amazon-corretto -y

```

## Install and configure PostgreSQL
Install PostgreSQL using yum:
```bash
sudo yum install postgresql postgresql-server 

```

Initialize a database:
```bash
postgresql-setup initdb
```


```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf
```

change METHOD to trust for localhost

Now start the server:
```bash
sudo service postgresql start
```


Log into the server:
```bash
sudo su - postgres
psql -U postgres
```

Create database and user:
```bash
postgres=# create database ticketmonster;
postgres=# create user ticketmonster with encrypted password 'ticketmonster';
postgres=# grant all privileges on database ticketmonster to ticketmonster;
```

Verify database:
```bash
postgres=# \l
```

Use \q to leave PostgreeSQL's command line utility

## Install and configure Wildfly 18
Switch to ec2-user and enter home directory
```bash
cd ~
```

Download:
```bash
wget https://download.jboss.org/wildfly/18.0.0.Final/wildfly-18.0.0.Final.tar.gz
```

Extract:
```bash
tar xzf wildfly-18.0.0.Final.tar.gz
```

Adjust configuration:
```bash
nano wildfly-18.0.0.Final/standalone/configuration/standalone.xml
```

Search for:
```xml
<interfaces>
    <interface name="management">
        <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
    </interface>
    <interface name="public">
        <inet-address value="${jboss.bind.address:127.0.0.1}"/>
    </interface>
</interfaces>
```

And replace "127.0.0.1" with "0.0.0.0" to bind on all IPv4 addresses on the local machine.

Run standalone.sh
```bash
cd wildfly-18.0.0.Final/bin/
nohup ./standalone.sh &
```

Navigate to http://{PublicIp}:8080 to verify Wildfly is up and running.

Enable admin user (still under wildfly-18.0.0.Final/bin/):
```bash
./add-user.sh
```

Enter
- a
- admin
- b


Set password for admin user (still under wildfly-18.0.0.Final/bin/):
```bash
./add-user.sh
```

Enter:
- a
- admin
- a
- admin
- yes
- admin
- PowerUser
- no

Navigate to http://{PublicIp}:9990 

Add PostgreSQL Pool to datasources
```bash
cd ~
mkdir legacy
cd legacy
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/legacy/commands.cli
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/legacy/postgresql-9.4-1202.jdbc41.jar
~/wildfly-18.0.0.Final/bin/jboss-cli.sh -c --file=commands.cli
```

Review configuration in Management Console {PublicIp}:9990

## Deploy TicketMonster

Download and deploy ticketmonster app

```bash
cd ~/wildfly-18.0.0.Final/standalone/deployments/
wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/legacy/ticket-monster.war
touch ticket-monster.war.dodeploy 
```

Navigate to http://{PublicIp}:8080/ticket-monster
