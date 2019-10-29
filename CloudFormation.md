# CloudFormation
[Working phase 04.01]
## Template Anatomy
A template is a JSON- or YAML-formatted text file that describes your AWS infrastructure. The following examples show an AWS CloudFormation template structure and its sections.

```yaml
AWSTemplateFormatVersion: "version date"

Description:
  String

Metadata:
  template metadata

Parameters:
  set of parameters

Mappings:
  set of mappings

Conditions:
  set of conditions

Transform:
  set of transforms

Resources:
  set of resources

Outputs:
  set of outputs
```

## Resources

## Template
```yaml
AWSTemplateFormatVersion: 2010-09-09

Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
  SubnetIdA:
    Type: AWS::EC2::Subnet::Id
  SubnetIdB:
    Type: AWS::EC2::Subnet::Id
  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName

Mappings: 
  RegionMap: 
    eu-central-1: 
      AMI: "ami-00aa4671cbf840d82"
    eu-west-1: 
      AMI: "ami-0ce71448843cb18a1"
    us-east-1: 
      AMI: "ami-00c03f7f7f2ec15c3"

Resources:

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Open database for access
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - FromPort: 22
          IpProtocol: tcp
          ToPort: 22
          CidrIp: 0.0.0.0/0

  Ec2Instance: 
    Type: AWS::EC2::Instance
    Properties: 
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      InstanceType: t2.medium
      KeyName: !Ref KeyPairName
      NetworkInterfaces: 
        - AssociatePublicIpAddress: true
          DeviceIndex: "0"
          GroupSet: 
            - !Ref InstanceSecurityGroup
          SubnetId: !Ref SubnetIdA
      Tags:
       - Key: Name
         Value: CF-Instance 

```


## Launch CloudFormation Stack
1. Navigate to the AWS CloudFormation Console
1. Click *Create Stack*
1. Upload your CloudFormation template
    ![](img/cf_01.JPG)
1. Specify stack details
    ![](img/cf_02.JPG)
    *exemplary parameters*
1. Skip stack options
1. Review and under Capabilities select *I acknowledge that AWS CloudFormation might create IAM resources.*
1. Click *Create Stack*
1. Review Stack Creation Events in the AWS CloudFormation Console

## Add Output

### Resources
- https://theburningmonk.com/cloudformation-ref-and-getatt-cheatsheet/

### Tasks
1. Add ec2 instance id as output
1. Add ec2 instance ip as output
1. Update Stack and review changes

## Add Condition

### Resources
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html

### Tasks
1. Add parameter DeploymentType with allowed values *Dev* and *Prod*
1. Add conditon to use a bigger ec2 instance if *Prod* is selected
1. Update Stack and review changes

## Adjust Security Group

### Resources
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group-rule-1.html

### Tasks
1. allow inbound connection on 8080
1. allow inbound connection on 9990
1. Update Stack and review changes

## Add User Data to EC2 Instance
### Resources
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html#cfn-ec2-instance-userdata

### Tasks
1. Add User Data Script to EC2 Instance
```yaml
      UserData:
        'Fn::Base64': !Sub |
          #!/bin/bash
          echo ${AWS::Region}
          echo ${AWS::StackName}
          sudo yum update -y
          sudo yum upgrade -y
          sudo yum autoremove -y
          sudo yum install java-11-amazon-corretto -y
          cd ~
          wget https://download.jboss.org/wildfly/18.0.0.Final/wildfly-18.0.0.Final.tar.gz
          tar xzf wildfly-18.0.0.Final.tar.gz
          sed 's/jboss.bind.address.management:127.0.0.1/jboss.bind.address.management:0.0.0.0/g' wildfly-18.0.0.Final/standalone/configuration/standalone.xml > wildfly-18.0.0.Final/standalone/configuration/tmp_standalone.xml
          sed 's/jboss.bind.address:127.0.0.1/jboss.bind.address:0.0.0.0/g' wildfly-18.0.0.Final/standalone/configuration/tmp_standalone.xml >  wildfly-18.0.0.Final/standalone/configuration/standalone.xml
          mkdir dockerize
          cd dockerize
          wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/cloudinit/commands.cli
          wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/cloudinit/postgresql-9.4-1202.jdbc41.jar
          wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/cloudinit/customize.sh
          mkdir -p /opt/tmp/
          mv postgresql-9.4-1202.jdbc41.jar /opt/tmp/postgresql-9.4-1202.jdbc41.jar
          chmod +x customize.sh
          ./customize.sh
          cd ~/wildfly-18.0.0.Final/standalone/deployments/
          wget https://materialien.s3.eu-central-1.amazonaws.com/workshop/cloudinit/ticket-monster.war
          touch ticket-monster.war.dodeploy 
          cd ~/wildfly-18.0.0.Final/bin/
          nohup ./standalone.sh &
```
1. Delete Stack and Create new Stack to view changes
1. SSH to the instance and check the cloud-init-output.log
```bash
tail -f /var/log/cloud-init-output.log 
```
1. Navigate to the public ip of the server on port 8080 and 8080/ticket-monster/

## Add RDS Instance

## Resources
- AWS::RDS::DBInstance https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html
- AWS::EC2::SecurityGroup https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html
- AWS::RDS::DBSubnetGroup https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html

### Tasks
1. Add Parameters
  - DBPassword
  - DBUser
1. Add Resources

```yaml
  DBEC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Open database for access
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - FromPort: 5432
          IpProtocol: tcp
          ToPort: 5432
          SourceSecurityGroupId: !Ref InstanceSecurityGroup

  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: subnet group
      SubnetIds:
        - !Ref SubnetIdA
        - !Ref SubnetIdB

  DBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBName: ticketmonster
      Engine: postgres
      EngineVersion: 9.6.8
      Port: "5432"
      MultiAZ: false
      MasterUsername: !Ref DBUser
      DBInstanceClass: db.t2.micro
      AllocatedStorage: 20
      MasterUserPassword: !Ref DBPassword
      VPCSecurityGroups:
        - !GetAtt DBEC2SecurityGroup.GroupId
      DBSubnetGroupName: !Ref DBSubnetGroup
      StorageEncrypted: false
```

1. Add Environment Variables to User Data
```bash
export DB_HOST=${DBInstance.Endpoint.Address}
export DB_PORT=${DBInstance.Endpoint.Port}
export DB_NAME=postgres
export DB_USERNAME=${DBUser}
export DB_PASSWORD=${DBPassword}
```
1. Delete Stack and Create new Stack to view changes
1. SSH to the instance and check the cloud-init-output.log
```bash
tail -f /var/log/cloud-init-output.log 
```
1. Navigate to the public ip of the server on port 8080 and 8080/ticket-monster/

## Add Custom Resource

## Resources
- [CloudFormation Custom Resource](CloudFormationCustomResource)

### Tasks
1. Adjust the CloudFormation template to use a generated Password from the Custom Resource Example
1. Replace the DBPassword Parameter by the Custom Resource
