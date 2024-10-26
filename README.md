# Certificate validation for godaddy in aws
1. Go certificate manager in same region and 
    wirite your domine name in this format *.(your domine name)
                    example = *.ramavath.xyz
2. select DNS validation method 
3. select key algorithm RSA 2048 and give tag name if you want and hit create certification
4. Click on your certificate and copy c_name name and c_name value
5. Go to Godaddy and go your product and click manage domine and add records 
    copy the c_name name only like(_75edfdbb818fa571e229cc207148e291.mhbtsbpdnt) 
    c_name value(_75edfdbb818fa571e229cc207148e291.mhbtsbpdnt.acm-validations.aws)
  


# Steps to achieve CI CD in AWS
1. In AWS create security groups for
    1.ELB
      * HTTP - anywhere
      * HTTPS - anywhere
      * ssh - myip
    2. Tomcat
      * 8080 - ELBSG
      * ssh - myip
      * 8080 - myip
    3.Backend services
      * MYSQL/AUORA(3306) - TomcatSG
      * MEMCACHE 11211 - TomcatSG
      * rabbitmq 5672 - TomcatSG
      * alltraffic - BackendSG
      * ssh - myip
2. Create instances for 
    1. mysql almalinux (CentOS Stream 9) (x86_64) instance which is  available in amis market place and attach the bash file which provided in userdata
    2. memcache almalinux (CentOS Stream 9) (x86_64) instance which is  available in amis market place and attach the bash file which provided in userdata
    3. rabbitmq almalinux (CentOS Stream 9) (x86_64) instance which is  available in amis market place and attach the bash file which provided in userdata
    4. tomcat instance ubuntu 22
    5. Now ssh each instances and check user script working or not by using these two methods
       *. ss -tunlp | grep <port number>
           ss -tunlp | grep 3306  for mysql
       *. systemctl status mariadb
               both works
    6. Tomcat home directory for local is (usr/local/tomcat) for virtual machine(var/lib/tomcat)
     
3. Create the private hosted zones in route 53 select the same regin where you created the vm's
   Create the records for mysql, memcache, and rabitmq with their private ips (simple routing -> simple record) in 
5. Now build the app locally using mvn install
    * Install maven, java, awscli locally 
   i. Make changes in application property file which is located in (src/main/resourcesj/application.properties)
       change the host names which is created in the previous classes i.e db01.vprofile.in , mc01.vprofile.in, and rmq.vprofile.in finally save .
6. Build artifact using mvn clean install

7. Create user  with s3 full access in IAM
  * aws configure
  * create s3 bucket and upload the war file from local to s3
      aws s3 mb s3://vpro-bucket
      aws s3 cp target/vprofile-v2.war s3://vpro-bucket
8. Create iam role with s3 full access and attach this role to tomcat instance and ssh instance and copy the war file from s3 bucket to tomcat instance
  * aws s3 cp s3://vpro-bucket/vprofile-v2.war /tmp/
  * stop tomcat service
    systemctl stop tomcat9
  * then remove
    rm -rf /var/lib/tomcat9/webapps/ROOT
  * and copy
  cp /tmp/vprofile-v2.war /var/lib/tomcat9/webapps/ROOT.war
  * and start tomcat service
    systemctl start tomcat9
      * for conformation 
      cd /var/lib/tomcat9/webapps/ROOT/WEB_NF/classes/appplication.properties
9. create the Load Balancer
    For that you have to create the target group
    -> Select instances
    -> Protoco http and port 8080
    -> Health check protocol /login
    -> advanced health check port 8080 hit next 
    -> select tomcat instance and click include as pending below hit create target group
  * go to loadbalancer
    -> Select application loadbalancer keep the interner facing and ipv4
    -> Select atleast min 3 availability zones
    -> Select load balancer security group which we created
    -> add listner HTTPS 
    -> In Secure listener settings selct our certificate 
    -> Click on create load balanceer 
    -> Go to load balancer copy the LB endpoint and go to the Godaddy and add DNS records with c_name (name= optional value= LB end point)
10. Create Auto scaling group 
  for that create Launch configuration
  *. Steps for launch configuration
    -> name your launch configure and write the discription for it
    -> go to my ami and select an image and select t2.micro
    -> select key pair and app security group and hit create lauch configure
  *. Steps for create auto scaling group
    ->select created launch configure
    ->Select all the subnets 
    -> Select desire and max instances
    ->Select Load balancer and enable monitoring
    ->
    -> 
    ->
