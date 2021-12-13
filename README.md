# WordPress Project
I created a WordPress site on a Ubuntu Server 20.04 LTS running in the us-east-1 (N.Virgina) region in AWS Cloud. I installed an apache2 webserver on my virtual server with MySQL, and PHP. I allocated an elastic IP address and registered a domain name using Route 53 (AWS) for the WordPress site. You can access it using - 3.220.165.196 or http://techwithbashiir.com/.  

I undertook this personal project to demonstrate my AWS and Linux system administration skills. The key goals of this project were that:  

- The WordPress site had to be highly available and operational at all times 

- It must meet the minimal performance requirements for reliability purposes


1. **Linux Server ** 
1a.Provisioning Ubuntu Server on AWS
The first step was to provision an ubuntu server on AWS to allow us to run our servers on using my free-tier account. I selected the N.Virgina (us-east-1) as my region and selected the Ubuntu 18.04.6 LTS as my AMI and the t.2 micro as my instance type. I kept the default settings for the storage and network configuration ( 8GB SSD gp2 and default VPC). I added a simple tag in order to recognise my resource.   

I also configured the security group of the instance to allow SSH, HTTP & HTTPS traffic into the server. This kept port 22, 80 and 443 open for incoming traffic from all hosts (0.0.0.0/0). Once done I launched my instance and it was up and running very quickly. 

<img width="1157" alt="Screenshot 2021-12-13 at 10 47 28" src="https://user-images.githubusercontent.com/89197223/145798935-c119fc55-abc4-4727-b1d6-ea67c79360b5.png">

1.B Associating elastic IP address with the Ubuntu server 
I went to Network & security under compute service in AWS and selected Elastic IP's. This was to allocate a fixed public IPV4 address that I can attach to my ubuntu server. It provided me the IP address - 3.220.165.196 which will be strictly for my instance, and it will not change in the event of failure or the instance being stopped until I manually release the address. This also simplifies management for me as I don't need to update any of my sessions with a new IP address when connecting to the remote server.

1.C Connecting to the remote server using SSH 
I downloaded the corresponding private key .pem to my local machine to use when connecting to the remote server using SSH. The key's were stored in my Downloads directory with default permission levels that were not secure. I modified the private key file to make groups and others have no permissions by running chmod 0700 against it. This ensured that only the authorised owner of the key had read and write access to the SSH key.  

To connect to my Ubuntu instance I started the terminal session and executed the command –             ssh -i privatekeyfile.pem@3.220.165.196. This connected me to my remote Ubuntu server on AWS and I was able to begin performing configurations on it.

1.C User creation and authentication  
This was pretty straightforward: 
- Ran adduser websiteuser  
- Added it to the sudoers group using usermod -aG sudo websiteuser; 
- Allowed it to use the SSH public key by copying it from root (and granted ownership of this copy) using rsync: rsync --archive --chown=lizfer:lizfer ~/.ssh /home/websiteuser. 

At this point, my newly-created user had administrative privileges whenever it used sudo and I could use this account to access from any remote client that had the SSH private key installed. 

 

 
