## Overview
This repository consists of two YAML files that can be used in AWS CloudFormation to deploy a PHP application into a scalable and highly availble (multi-AZ) LAMP stack, using an RDS managed database. The 'LAMP stack scalable and durable' template provided in the AWS documentation was used as a starting point and can be found here: [Application Frameworks] (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-appframeworks-eu-west-2.html).
However this sample template is set up to deploy a PHP application into the default VPC and subnets, whereas I wanted to be able to use a custom VPC. I also wanted the script to retrieve the application source code from a private GitHub repository, rather than simply creating a sample index.php file. The steps taken to achieve this, along with general explanations of the code, are covered in the Key Developments section.
## How to Use
1. Run NetworkingComponents.yaml script using AWS CloudFormation and wait for it to complete.
2. Run the LAMPStack.yaml script using AWS CloudFormation and fill in the required parameters.
  - The name of a private GitHub repository containing the source code of your PHP application is required, along with your Github credentials to allow Cloudformation to pull the code. Note that a config.php file containing the database credentials is created by the script so should not be within the GitHub repository.
  - Your local IP address should be given for both MySQLClientLocation and SSHLocation to limit connectivity.
3. When the stack completes, navigate to 'Outputs' and use the provided URL to access the appliction.
## Key Developments
### Security Group Fix
Firstly, I tested the sample template (linked above) without making any changes, using the default VPC / Subnets. The stack ran successfully however following the link to the web app, which is provided as an output, results in the page timing out. This is indicative of not having the correct security group permissions. The problem (at the time of writing) was that the web server security group was set up to only allow http traffic from the Application Load Balancer (ALB), which didn't have any security groups attached so wouldn't accept any traffic. Therefore, the security group 'ALBSecurityGroup' was added so that the ALB would accept http traffic from anywhere.
### Creating a custom VPC & Subnets
The file NetworkingComponents.yaml will create a new VPC with two subnets, into which the application can be deployed. Note that the code within the metadata scetions is automatically created by cloudformation and is only used to provide a graphic of the components while using the designer. Further details are stated below.
- The subnets (PublicSubnet1 & PublicSubnet2) are in different availability zones with non-overlapping CIDR blocks, which is needed to facilitate load balancing by the ALB.
- The subnets must both have the MapPublicIpOnLaunch property set to true, so that instances launched within them receive a public IPv4 address.
- The VPC required an InternetGateway and corresponding VPCGatewayAttachment to facilitate any connections over the internet.
- A PublicRouteTable with a PublicRoute and corresponding subnet associations were required in order to provide internet connectivity for the subnets.
### Changes to the sample template
To allow the script to run using the VPC & subnets created by NetworkingComponents.yaml, the following changes were made:
- The database requires a subnet group to be specified to allow multi-AZ deployment. A subnet group was therefore added (DBSubnetGroup), consisting of whichever subnets had been chosen. 
