# AWS Cloud Solution For 2 Company's Websites Using A Reverse Proxy Technology

WARNING: This infrastructure set up is NOT covered by AWS free tier. Therefore, ensure to DELETE ALL the resources created immediately after finishing the project. Monthly cost may be shockingly high if resources are not deleted. Also, it is strongly recommended to set up a budget and configure notifications when your spendings reach a predefined limit. Watch this video to learn how to configure AWS budget.

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website tooling for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

# Starting Off AWS Project

1. Properly configure your AWS account and Organization Unit Watch How To Do This here
   
- Create an AWS Master account. (Also known as Root Account)

- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there) Move the DevOps account into the Dev OU.

- Login to the newly created AWS account using the new email address.

- Create a free domain name for your fictitious company at Freenom domain registrar here.

- Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that by clicking on these links below

  https://www.youtube.com/watch?v=hRSj2n-XKGM

  https://www.youtube.com/watch?v=IjcHp94Hq8A

  ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ae2ede73-03ce-4af1-9c2e-575182b847bf)

NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

- Project:<Give your project a name>

- Environment:<dev>

- Automated: (If you create a recource using an automation tool, it would be )
 
 # Setting Up Infrastucture

 1. Create a VPC

 ![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c3fa559b-8e4e-4aaa-aaa4-13605712f062)

 2. Create subnets as shown in the architecture

 ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8a212d26-1108-4997-bd1f-57287fb1a104)

 3. Create a route table for the public subnets

 4. Create a route table for the private subnets

 5. Create an Internet Gateway and attach it to VPC

 6. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

 7. Edit routes of both public and private route-table and add Intenet Gateway and Nat Gateway to them

 8. Edit Subnet Associations of both Public and Private route-table and associate public and private subnets to them

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/851ea247-655d-46bc-ac31-69f933a3f34f)

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c8bd70ca-98bf-4ecd-9786-d2d022df766a)

9. Create a Security Group for:
    
 - Application Load Balancer: ALB will be available from the Internet.

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/19474dad-1af8-4337-907d-4faa588a7e20)

10. - Bastion Servers:  Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bf10959f-cad5-4713-a02b-428fc4fe3d33)

11. - Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). Alt text
   
      ![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/df914b40-0369-4ead-a97f-9d2e10da33a4)

12. - Internal Application Load Balancer: ALB will be available from the Nginx servers.

    ![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a123b3a4-4a6c-45ba-b94b-3a101d4caa30)

13. - Webservers: Access to Webservers should only be allowed from the Internal Application Load Balancer. Alt text

      ![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/41c90be8-c4f9-4a7f-8a76-cf732c619f89)

14. - Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

    ![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0f720e4e-29f4-4020-b1b5-6272e20c7908)















  
