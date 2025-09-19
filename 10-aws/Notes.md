1. Explian how will you design a highly available and scalable multi-tier app.
As a firt step-> Create **VPC** [I will interact with developers to understand number of appl's that are going to be deployed / number of applications that are goign to us ethis VPC]
depending upon that, I will define a **CIDR** block for the VPC.
-> within VPC, i will create **public** and **private** subnets and also define **CIDR range for both public and provate subnets** accordingly.
-> with in public subnet, I will place load balancer
-> with in private subnet, I will deploy virtual machines and these virtual machines will be configured through **AWS ASG(Auto scaling group)**, so that these VM's can be scalable.
   with in **VM's I will deploy application server where application will be deployed**. These applications on VM's interact with **DB** which I am going to deploy on the private subnet.
   if required, I will create a saperate private subnet for the DB, it totally depends on the level of isolation/ I can use a managed DB instance according to the requirement from the deveploers.  
   
  Now, after created VPC as explianed above,
  any **external user** can access -> **load balancer** (will also be grated access to the private subnets- here load balancer will be a public/private subnet VPC load balancer), this load balancer assigned with target groups / ASG (as VM's in the private subnet) ->
  request fro Load balancer **goes to the application server in the private subnet**,**from where application interacts with DB fetch the require information and returns information back to the client.** 

  Note: avaliability and scalability here is achieved here using AWS ASG.
        security by defining right size of CIDR block and application and DB defined in the private subnets / if required I will use managed AWS DB servuces like RDS.
   <img width="302" height="430" alt="image" src="https://github.com/user-attachments/assets/0b9c1af9-d7d4-4b69-a708-ed40763293da" />
   
2. What is AWS NAT(Network Address Translation) and when is it used?
In my current company, we have VM which as application running in the prvate subnet, where these backend application need to download some dependencies from GITHUB.
because thes applications deployed in the private subnet. Obviously they don't have external access or access to the internet. So in these cases, to allow the applications talk to internet or talk to GitHub to download packages,
we create a Nat gateway. So Nat in AWS performs network address translation. So we create a Nat gateway and create a route in AWS where the route will have a route table, which has the destination route
0.0.0.0 pointing to the Nat gateway instead of internet gateway.

**Application / any process in the application tries to download something from internet -> request sent to -> NAT, performs Network Address Translation-> through NAT request sent to GITHUB.**

Note: NAT will change the source IP address (application server IP) to the NAT public IP and sends request to the destination address i.e here it is GITHUB.

This way source IP of the application is not exposed.
Client request -> OSI Layer -> L4 (client request is converted into packets, packet will have source IP, Dest IP and all other information, so thet your packet actually know how to travel from where to where)-> here NAT will change same packet where client request is converted into packet at L4,change application source IP to NAT's public IP and send request to the public internet. here 
destination will not respond to the source application server IP instead forwards it to the NAT public IP. This way Application Server IP in the private subnet is not exposed to the external world.




   
   
     

        

  
