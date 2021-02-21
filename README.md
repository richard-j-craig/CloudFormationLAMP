## Overview
This repository consists of two YAML files that can be used in AWS CloudFormation to deploy a PHP application into a scalable and highly available (multi-AZ) LAMP stack, using an RDS managed database. The 'LAMP stack scalable and durable' template provided in the AWS documentation was used as a starting point and can be found here: [Application Frameworks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-appframeworks-eu-west-2.html).
However, this sample template is set up to deploy a PHP application into the default VPC and subnets, whereas I wanted to be able to use a custom VPC. I also wanted the script to retrieve the application source code from a private GitHub repository, rather than simply creating a sample index.php file. The steps taken to achieve this, are covered in the Key Developments section. Note that the code within the metadata sections for both YAML files was automatically created by CloudFormation and is only used to provide a graphic of the components in the visual designer.
## How to Use
1. Run NetworkingComponents.yml script using AWS CloudFormation and wait for it to complete.
2. Run the LAMPStack.yml script using AWS CloudFormation and fill in the required parameters.
  - The name of a private GitHub repository containing the source code of your PHP application is required, along with your GitHub credentials to allow CloudFormation to pull the code. Note that a config.php file containing the database credentials is created by the script so should not be within the GitHub repository. Other files querying the database should then reference this config file to make the connection.
  - Your local IP address should be given for both MySQLClientLocation and SSHLocation to limit connectivity.
3. When the stack completes, navigate to 'Outputs' and use the provided URL to access the application.
## Key Developments
### Security Group Fix
Firstly, I tested the sample template (linked above) without making any changes, using the default VPC / Subnets. The stack ran successfully however following the link to the web app, which is provided as an output, results in the page timing out. This is indicative of not having the correct security group permissions. The problem (at the time of writing) was that the web server security group was set up to only allow http traffic from the Application Load Balancer (ALB), which didn't have any security groups attached so wouldn't accept any traffic. Therefore, the security group 'ALBSecurityGroup' was added so that the ALB would accept http traffic from anywhere.
### Creating a custom VPC & Subnets
The file NetworkingComponents.yml will create a new VPC with two subnets, into which the application can be deployed. Further details are stated below.
- The subnets (PublicSubnet1 & PublicSubnet2) are in different availability zones with non-overlapping CIDR blocks, which is needed to facilitate load balancing by the ALB.
- The subnets must both have the MapPublicIpOnLaunch property set to true, so that instances launched within them receive a public IPv4 address.
- The VPC required an InternetGateway and corresponding VPCGatewayAttachment to facilitate any connections over the internet.
- A PublicRouteTable with a PublicRoute and corresponding subnet associations were required in order to provide internet connectivity for the subnets.
### Changes to the sample template
To allow the script to run using the VPC & subnets created by NetworkingComponents.yml, the following changes were made:
- The database requires a subnet group to be specified to allow multi-AZ deployment. A subnet group was therefore added (DBSubnetGroup), consisting of whichever subnets had been chosen.

To allow the script to use application source code from a private GitHub repository, the following changes were made:
- Added paramters for the user to enter their GitHub username, password / tocken and chosen repository (GithubUsername, GithubPassword, GithubRepo).
- Added an AWS::CloudFormation::Authentication section within the LaunchConfig, to authenticate cfn to GitHub.
- Added a source section to the LaunchConfig to import the GitHub code as a tarball.
- Made a config.php file to be created within the LaunchConfig, to be used by the PHP application for connections to the database.

Other changes:
- I wanted to be able to connect the database via MySQL Workbench from my laptop to provide a GUI. To allow this, a security group ingress rule was added to DBSecurityGroup to allow connections over port 3306 from a given IP address (set by the MySQLClientLocation parameter) and the PublicAccessibility property for MySQLDatabase was set to true, without this property no connections can be made from outside the VPC.
  - MySQL Workbench can easily be used to import data from an existing database using an SQL dump file.
- Options for instance types and corresponding mappings were reduced for readability.
