AWSTemplateFormatVersion: '2010-09-09' Description: > AWS CloudFormation Template to set up VPC with 2 Public and 3 Private Subnets, NAT Gateway, Route Tables, and EC2 Instances for Jenkins, SonarQube, and Nexus.

Parameters: EnvironmentName: Type: String Description: Name of the environment (e.g., Dev, Prod)

VpcCIDR: Type: String Description: CIDR block for the VPC

PublicSubnet1CIDR: Type: String Description: CIDR block for Public Subnet 1

PublicSubnet2CIDR: Type: String Description: CIDR block for Public Subnet 2

PrivateSubnet1CIDR: Type: String Description: CIDR block for Private Subnet 1

PrivateSubnet2CIDR: Type: String Description: CIDR block for Private Subnet 2

PrivateSubnet3CIDR: Type: String Description: CIDR block for Private Subnet 3

InstanceType: Type: String Default: t3.medium Description: EC2 instance type

ImageId: Type: AWS::EC2::Image::Id Description: AMI ID to use for EC2 instances

Resources: VPC: Type: AWS::EC2::VPC Properties: CidrBlock: !Ref VpcCIDR EnableDnsSupport: true EnableDnsHostnames: true Tags: - Key: Name Value: !Sub "${EnvironmentName}-VPC"

InternetGateway: Type: AWS::EC2::InternetGateway Properties: Tags: - Key: Name Value: !Sub "${EnvironmentName}-IGW"

AttachGateway: Type: AWS::EC2::VPCGatewayAttachment Properties: VpcId: !Ref VPC InternetGatewayId: !Ref InternetGateway

PublicSubnet1: Type: AWS::EC2::Subnet Properties: VpcId: !Ref VPC CidrBlock: !Ref PublicSubnet1CIDR AvailabilityZone: !Select [0, !GetAZs ''] MapPublicIpOnLaunch: true Tags: - Key: Name Value: !Sub "${EnvironmentName}-PublicSubnet-1"

PublicSubnet2: Type: AWS::EC2::Subnet Properties: VpcId: !Ref VPC CidrBlock: !Ref PublicSubnet2CIDR AvailabilityZone: !Select [1, !GetAZs ''] MapPublicIpOnLaunch: true Tags: - Key: Name Value: !Sub "${EnvironmentName}-PublicSubnet-2"

PrivateSubnet1: Type: AWS::EC2::Subnet Properties: VpcId: !Ref VPC CidrBlock: !Ref PrivateSubnet1CIDR AvailabilityZone: !Select [0, !GetAZs ''] Tags: - Key: Name Value: !Sub "${EnvironmentName}-PrivateSubnet-1"

PrivateSubnet2: Type: AWS::EC2::Subnet Properties: VpcId: !Ref VPC CidrBlock: !Ref PrivateSubnet2CIDR AvailabilityZone: !Select [1, !GetAZs ''] Tags: - Key: Name Value: !Sub "${EnvironmentName}-PrivateSubnet-2"

PrivateSubnet3: Type: AWS::EC2::Subnet Properties: VpcId: !Ref VPC CidrBlock: !Ref PrivateSubnet3CIDR AvailabilityZone: !Select [2, !GetAZs ''] Tags: - Key: Name Value: !Sub "${EnvironmentName}-PrivateSubnet-3"

PublicRouteTable: Type: AWS::EC2::RouteTable Properties: VpcId: !Ref VPC Tags: - Key: Name Value: !Sub "${EnvironmentName}-PublicRouteTable"

DefaultRoute: Type: AWS::EC2::Route Properties: RouteTableId: !Ref PublicRouteTable DestinationCidrBlock: 0.0.0.0/0 GatewayId: !Ref InternetGateway

PublicSubnet1RouteTableAssociation: Type: AWS::EC2::SubnetRouteTableAssociation Properties: SubnetId: !Ref PublicSubnet1 RouteTableId: !Ref PublicRouteTable

PublicSubnet2RouteTableAssociation: Type: AWS::EC2::SubnetRouteTableAssociation Properties: SubnetId: !Ref PublicSubnet2 RouteTableId: !Ref PublicRouteTable

EIP: Type: AWS::EC2::EIP Properties: Domain: vpc

NATGateway: Type: AWS::EC2::NatGateway Properties: AllocationId: !GetAtt EIP.AllocationId SubnetId: !Ref PublicSubnet1 Tags: - Key: Name Value: !Sub "${EnvironmentName}-NATGW"

PrivateRouteTable: Type: AWS::EC2::RouteTable Properties: VpcId: !Ref VPC Tags: - Key: Name Value: !Sub "${EnvironmentName}-PrivateRouteTable"

PrivateDefaultRoute: Type: AWS::EC2::Route Properties: RouteTableId: !Ref PrivateRouteTable DestinationCidrBlock: 0.0.0.0/0 NatGatewayId: !Ref NATGateway

PrivateSubnet1RouteTableAssociation: Type: AWS::EC2::SubnetRouteTableAssociation Properties: SubnetId: !Ref PrivateSubnet1 RouteTableId: !Ref PrivateRouteTable

PrivateSubnet2RouteTableAssociation: Type: AWS::EC2::SubnetRouteTableAssociation Properties: SubnetId: !Ref PrivateSubnet2 RouteTableId: !Ref PrivateRouteTable

PrivateSubnet3RouteTableAssociation: Type: AWS::EC2::SubnetRouteTableAssociation Properties: SubnetId: !Ref PrivateSubnet3 RouteTableId: !Ref PrivateRouteTable

InstanceSecurityGroup: Type: AWS::EC2::SecurityGroup Properties: GroupDescription: Allow HTTP/HTTPS for UI access VpcId: !Ref VPC SecurityGroupIngress: - IpProtocol: tcp FromPort: 80 ToPort: 80 CidrIp: 0.0.0.0/0 - IpProtocol: tcp FromPort: 443 ToPort: 443 CidrIp: 0.0.0.0/0 Tags: - Key: Name Value: !Sub "${EnvironmentName}-InstanceSG"

JenkinsInstance: Type: AWS::EC2::Instance Properties: InstanceType: !Ref InstanceType ImageId: !Ref ImageId SubnetId: !Ref PrivateSubnet1 SecurityGroupIds: - !Ref InstanceSecurityGroup Tags: - Key: Name Value: !Sub "${EnvironmentName}-Jenkins"

SonarQubeInstance: Type: AWS::EC2::Instance Properties: InstanceType: !Ref InstanceType ImageId: !Ref ImageId SubnetId: !Ref PrivateSubnet2 SecurityGroupIds: - !Ref InstanceSecurityGroup Tags: - Key: Name Value: !Sub "${EnvironmentName}-SonarQube"

NexusInstance: Type: AWS::EC2::Instance Properties: InstanceType: !Ref InstanceType ImageId: !Ref ImageId SubnetId: !Ref PrivateSubnet3 SecurityGroupIds: - !Ref InstanceSecurityGroup Tags: - Key: Name Value: !Sub "${EnvironmentName}-Nexus"

Outputs: VPCId: Description: VPC ID Value: !Ref VPC

JenkinsPublicAccess: Description: Jenkins UI Value: !GetAtt JenkinsInstance.PublicDnsName

SonarQubePublicAccess: Description: SonarQube UI Value: !GetAtt SonarQubeInstance.PublicDnsName

NexusPublicAccess: Description: Nexus UI Value: !GetAtt NexusInstance.PublicDnsName

