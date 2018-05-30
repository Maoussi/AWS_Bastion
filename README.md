# AWS_Bastion
Creation of a bastion to access a private instance in a VPC private subnet

1. Create 2 key pairs 
One will be used to connect to the bastion, the other one to connect to the private instance
In AWS Service -> EC2 -> Key pairs -> Create key pairs -> Save them in a chosen directory

2. Create a VPC with 2 subnets, one public and one private in one availability zone (AZ)
AWS Service -> VPC -> VPC wizard -> VPC with public and private subnets
CIDR IPv4: 10.0.0.0/16
Name of the VPC: Maoussi VPC
CIDR IPv4 of public subnet: 10.0.1.0/24
AZ: eu-west-1a
Name of public VPC: Public VPC
CIDR IPv4 of private VPC: 10.0.3.0/24
AZ: eu-west-1a
Name of private VPC: Private VPC
Use a NAT instance instead: t2.micro
Name of key pair: Choose one of the 2 key pairs created previously
Create VPC

3. Create a security group for the VPC to let
Security group -> Create security group 
               -> Tag: MaoussiSecurityGp
               -> Name: MaoussiSecurityGroup
               -> Description: Enable HTTP/ICMP
               -> VPC: Maoussi VPC
Inbound rules: Edit -> type: HTTP, port(80), source 0.0.0.0/0
                    -> Add new rule: type: HTTPS, port(443), source 0.0.0.0/0
		    -> Add new rule: type: tous les ICMP IPv4, source 0.0.0.0/0

4. Create the bastion in public subnet
In AWS Service -> Instances -> Launch an instance
AMI -> Windows server 2016 Base
Type -> t2.micro
Configure instance : network -> Maoussi VPC
		                Subnet -> Public Subnet
		                Auto-assign public IP: Enable
Add tags: key -> Name ;  Value: Bastion
Configure Security Group: Name: BastionSecurityGp ; Description: BastionSecurityGp created 2018-05-26T17:32:03.168+02:00
type: RDP ; port 3389 ; source 0.0.0.0/0
type: tous les ICMP IPv4, source 0.0.0.0/0
Launch instance
Go back to BastionSecurityG: -> inbound rules: type: RDP ; port 3389 ; source 0.0.0.0/0
 					       type: tous les ICMP IPv4, source 0.0.0.0/0
                             -> outbound rules: All traffic

5. Create an instance in the private subnet and add connectivity with the bastion
In AWS Service -> Instances -> Launch an instance
AMI: Windows server 2016 Base
Type: t2.micro
Configure instance : network -> Maoussi VPC
		                Subnet -> Private Subnet
		                Auto-assign public IP: Disable
Add tags: key -> Name ;  Value: My Instance
Configure Security Group: Name: MyInstSecurityGp ; Description: MyInstSecurityGp created 2018-05-26T17:32:03.168+02:00
Type: RDP ; port 3389 ; source sg-Bastion (security group of the Bastion)
type: tous les ICMP IPv4, source 0.0.0.0/0
Launch instance
Go back to MyInstSecurityGp: -> inbound rules: type: RDP ; port 3389 ; source sg-Bastion
					       type: tous les ICMP IPv4, source 0.0.0.0/0
                             -> outbound rules: All the traffic
			
Check the security group of the NAT: it should allow the incoming traffic from the private instance
inbound rules: All traffic ; source sg-MyInstSecurityGp
outbound rules: All traffic ; source 0.0.0.0/0
                             
6. Connect to your instance
Select Bastion -> Connect -> get the password (Passsword 1) 
Select MyInstance -> Connect -> get the password (Passsword 2)
Select Bastion -> Connect through the RDP link: use Password 1
Once in the Bastion, search Remote Desktop connection in the search tab
Use the private IP of your private instance to connect to it
Username : Administrator
Password : Password 2

That's it, your are in your private instance, using a bastion as a secure jump sever
You can try to ping www.google.com in the command line, it should give an answer.



          

