# Networking
```yaml
AWSTemplateFormatVersion: 2010-09-09

Parameters:
  DeploymentType:
    Type: String
    AllowedValues: [ dev, prod ]
    Default: dev
  AZA:
    Type: AWS::EC2::AvailabilityZone::Name
  AZB:
    Type: AWS::EC2::AvailabilityZone::Name


Conditions:
  ProdDeployment: !Equals [ !Ref DeploymentType, prod]

Resources:

    # VPC
    VPC:
      Type: AWS::EC2::VPC
      Properties:
        CidrBlock: 10.0.0.0/16
        EnableDnsSupport: true
        EnableDnsHostnames: true
        Tags:
          - Key: Application
            Value: !Ref AWS::StackId
          - Key: Network
            Value: Public

    InternetGateway:
      Type: AWS::EC2::InternetGateway
      Properties:
        Tags:
          - Key: Application
            Value: !Ref AWS::StackId
          - Key: Network
            Value: Public

    VPCGatewayAttachment:
      Type: AWS::EC2::VPCGatewayAttachment
      Properties:
        InternetGatewayId: !Ref InternetGateway
        VpcId: !Ref VPC

    PubSubnetA:
      Type: AWS::EC2::Subnet
      Properties:
        AvailabilityZone: !Ref AZA
        CidrBlock: 10.0.1.0/24
        VpcId: !Ref VPC

    PubSubnetB:
      Type: AWS::EC2::Subnet
      Properties:
        AvailabilityZone: !Ref AZB
        CidrBlock: 10.0.2.0/24
        VpcId: !Ref VPC

    PrivSubnetA:
      Type: AWS::EC2::Subnet
      Properties:
        AvailabilityZone: !Ref AZA
        CidrBlock: 10.0.5.0/24
        VpcId: !Ref VPC

    PrivSubnetB:
      Type: AWS::EC2::Subnet
      Properties:
        AvailabilityZone: !Ref AZB
        CidrBlock: 10.0.6.0/24
        VpcId: !Ref VPC

    PublicRouteTable:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC

    PublicRoute:
      Type: AWS::EC2::Route
      DependsOn: VPCGatewayAttachment
      Properties:
        RouteTableId: !Ref PublicRouteTable
        DestinationCidrBlock: 0.0.0.0/0
        GatewayId: !Ref InternetGateway

    SubnetRouteTableAssociationPub1:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        SubnetId: !Ref PubSubnetA
        RouteTableId: !Ref PublicRouteTable

    SubnetRouteTableAssociationPub2:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        SubnetId: !Ref PubSubnetB
        RouteTableId: !Ref PublicRouteTable

    NATEIPA:
     Type : AWS::EC2::EIP
     Properties:
        Domain: !Ref VPC

    NatGatewayPubA:
      Type: AWS::EC2::NatGateway
      DependsOn:
        - VPCGatewayAttachment
      Properties:
        AllocationId: !GetAtt NATEIPA.AllocationId
        SubnetId: !Ref PubSubnetA

    NATEIPB:
     Condition: ProdDeployment
     Type : AWS::EC2::EIP
     Properties:
        Domain: !Ref VPC

    NatGatewayPubB:
      Condition: ProdDeployment
      Type: AWS::EC2::NatGateway
      DependsOn:
        - VPCGatewayAttachment
      Properties:
        AllocationId: !GetAtt NATEIPB.AllocationId
        SubnetId: !Ref PubSubnetB

    PrivateRouteTableA:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC

    PrivateRouteToInternetA:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PrivateRouteTableA
          DestinationCidrBlock: 0.0.0.0/0
          NatGatewayId: !Ref NatGatewayPubA

    SubnetRouteTableAssociationA:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        SubnetId: !Ref PrivSubnetA
        RouteTableId: !Ref PrivateRouteTableA

    # AZ-B
    PrivateRouteTableB:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC

    PrivateRouteToInternetB:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PrivateRouteTableB
          DestinationCidrBlock: 0.0.0.0/0
          NatGatewayId: !If [ProdDeployment, !Ref NatGatewayPubB, !Ref NatGatewayPubA]

    SubnetRouteTableAssociationB:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        SubnetId: !Ref PrivSubnetB
        RouteTableId: !Ref PrivateRouteTableB

    S3Endpoint:
      Type: AWS::EC2::VPCEndpoint
      Properties:
        RouteTableIds:
          - !Ref PrivateRouteTableA
          - !Ref PrivateRouteTableB
        ServiceName: !Join ["", ["com.amazonaws.", !Ref "AWS::Region", ".s3"]]
        VpcId: !Ref VPC

Outputs:
  VpcId:
    Value: !Ref VPC
  PubSubnetA:
    Value: !Ref PubSubnetA
  PubSubnetB:
    Value: !Ref PubSubnetB
  PrivSubnetA:
    Value: !Ref PrivSubnetA
  PrivSubnetB:
    Value: !Ref PrivSubnetB
```