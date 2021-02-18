# AutoScaling

In this lab, we will build a load-balanced web application that can automatically scale out/in based on CPU utilization.

We will create load balancer, compute instance, instance configuration, and then configure auto scaling. We will then verify auto scaling feature as configured threshold on CPU is crossed.

Autoscaling enables you to automatically adjust the number of compute instances in an instance pool based on performance metrics, such as CPU utilization. This helps you provide consistent performance for your end users during periods of high demand, and helps you reduce your costs during periods of low demand.

You select a performance metric to monitor, and set thresholds that the performance metric must reach to trigger an autoscaling event. When system usage meets a threshold, autoscaling dynamically allocates resources in near-real time. As load increases, instances are automatically provisioned: the instance pool scales out. As load decreases, instances are automatically removed: the instance pool scales in.

Autoscaling relies on performance metrics that are collected by the Monitoring service. These performance metrics are aggregated into one-minute time periods and then averaged across the instance pool. When three consecutive values (that is, the average metrics for three consecutive minutes) meet the threshold, an autoscaling event is triggered.

A cooldown period between autoscaling events lets the system stabilize at the updated level. The cooldown period starts when the instance pool reaches a steady state. Autoscaling continues to evaluate performance metrics during the cooldown period. When the cooldown period ends, autoscaling adjusts the instance pool's size again if needed

## Steps
### 1) At First create Oracle OCI Login Credentials. It could be either Trial account or Pay-as-you-go account.
### 2) Create SSH key pair

 The SSH (Secure Shell) protocol is a method for secure remote login from one computer to another. SSH enables secure system administration and file transfers over insecure networks using encryption to secure the connections between endpoints. SSH keys are an important part of securely accessing Oracle Cloud Infrastructure compute instances in the cloud.

Create a set of SSH public key pair. This key will be used for compute instance SSH access. The key pair can be created through cloud shell or any available Linux machine. 

        # ssh-keygen
        Generating public/private rsa key pair.
        Enter file in which to save the key (/root/.ssh/id_rsa):
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:
        Your identification has been saved in /root/.ssh/id_rsa.
        Your public key has been saved in /root/.ssh/id_rsa.pub.
        The key fingerprint is:
        SHA256:2rVDTujOKBMWspEdFCKg8/omlPVhFOYRsZ/T+nDunHk root@terraform
        The key's randomart image is:
        +---[RSA 2048]----+
        |+ ..oB+          |
        |.. .+.o          |
        |o  o.+           |
        | o+.oo. o.       |
        |  ++o..+S.+      |
        | +. o. +o= .     |
        |o  . ..oo.+      |
        |... o  +* oE     |
        | o.  o. +B.      |
        +----[SHA256]-----+
        # ls -lrt /root/.ssh/
        total 12
        -rw-------. 1 root root  550 Jan 29 09:27 authorized_keys
        -rw-------. 1 root root 1679 Feb 17 06:56 id_rsa
        -rw-r--r--. 1 root root  396 Feb 17 06:56 id_rsa.pub

### 3) Create VCN (Virtual Cloud Network)

### 4) Create Load Balancer and Update Security List

### 5)  Configure instance pool and auto scaling
Go to the OCI console. From OCI services menu, under Compute, click Instances.

Click Create Instance. Fill out the dialog box:

        Name your instance: Enter a name
        Create in Compartment: Choose the same compartment you used to create the VCN
        Choose an operating system or image source: For the image, we recommend using the Latest Oracle Linux available.
Click Show Shape, Network and Storage Options:

Availability Domain: Select an availability domain (the default AD 1 is fine)

Shape: Choose appropriate chape. This workshop uses VM.Standard.E2.1 shape.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/4.JPG)

Under Configure Networking:

Virtual cloud network compartment: Select your compartment

Virtual cloud network: Choose the VCN you created in Step 1

Subnet Compartment: Choose your compartment.

Subnet: Choose the Public Subnet under Public Subnets

Use network security groups to control traffic : Leave un-checked

Assign a public IP address: Check this option
![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/5.JPG)

Boot Volume and Add SSH Keys

Boot Volume: Leave the default, uncheck values

Add SSH Keys: Choose 'Paste SSH Keys' and paste the Public Key saved in Lab 1.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/6.JPG)

Click Show Advanced Options.

Under Management

Initialization Script: Choose Paste cloud-init script and paste the below script. Cloud-init script will be executed at the first boot only to configure the instance.


        #cloud-config
        packages:
        - httpd
        - stress

        runcmd:
        - [sh, -c, echo "<html>Web Server IP `hostname --ip-address`</html>" > /var/www/html/index.html]
        - [firewall-offline-cmd, --add-port=80/tcp]
        - [systemctl, start, httpd]
        - [systemctl, restart, firewalld]


![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/7.JPG)


Click Create.

NOTE: If 'Service limit' error is displayed, choose a different shape, such as VM.Standard.E2.2 OR VM.Standard2.2 OR choose a different AD.

Wait for Instance to be in Running state. You can scroll down to Work Requests to check the process of provisioning.

Click Instance name. Click More Actions, then select Create Instance Configuration.
Fill out the dialog box:

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/8.JPG)

CREATE IN COMPARTMENT: Choose your compartment

INSTANCE CONFIGURATION NAME : Provide a name

Click Create Instance Configuration.

In the Instance Configuration page, Click Create Instance Pool.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/9.JPG)

A new dialog box will appear. This is used to create initial configuration of the instance pool, such as how many compute instance to create initially, VCN, and Availability domain the instance pool should be created in. Fill out the dialog box:

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/10.JPG.png)

CREATE IN COMPARTMENT: Choose your compartment

INSTANCE POOL NAME: Provide a suitable name

NUMBER OF INSTANCES: 0
(This is the number of computes that should be launched when the pool is created. We will start with no compute)

Click Next.

On the Configure Pool Placement page:

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/11.JPG.png)

AVAILABILITY DOMAIN: Choose the AD where you want to place instances (you can choose the AD 1 if in Multi AD region)

VIRTUAL CLOUD NETWORK COMPARTMENT: Choose VCN's compartment

VIRTUAL CLOUD NETWORK: Choose your VCN

SUBNET COMPARTMENT: Choose your compartment

SUBNET: Choose the Public Subnet

ATTACH A LOAD BALANCER: Select this option.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/12.JPG.png)

LOAD BALANCER COMPARTMENT: Choose your compartment

LOAD BALANCER: Choose the Load Balancer created earlier

BACKEND SET: Choose the first backend set

PORT: 80

VNIC: Leave the default

Click Next and then Create. Wait for Instance Pool to be in RUNNING state (turns green).

From the Instance Pool Details page, click More Actions and choose Create Autoscaling Configuration.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/13.JPG)

On the Add Basic Details page:

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/14.JPG)

COMPARTMENT: Choose your compartment

AUTOSCALING CONFIGURATION NAME : Provide a name

INSTANCE POOL : This should show your instance pool name created earlier
Note: If the instance pool name is blank, try refreshing the browser and trying again.

Click Next.

On the Configure Autoscaling Policy page:

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/15.JPG)

Make sure that Metric-based Autoscaling is selected.

AUTOSCALING POLICY NAME : Provide a name

COOLDOWN IN SECONDS : 300 (This is he minimum period of time between scaling actions.)

PERFORMANCE METRIC : CPU utilization (This is the Metric to use for triggering scaling actions.)

SCALE-OUT OPERATOR : Greater than (>)

THRESHOLD PERCENTAGE : 10

NUMBER OF INSTANCES TO ADD : 1

SCALE-IN OPERATOR : Less than (<)

THRESHOLD PERCENTAGE : 5

NUMBER OF INSTANCES TO REMOVE : 1

MINIMUM NUMBER OF INSTANCES : 1 (this is the minimum number of instances that the pool will always have)

MAXIMUM NUMBER OF INSTANCES : 2 (this is the maximum number of instances that the pool will always have)

INITIAL NUMBER OF INSTANCES : 1 (this is how many instances will be created in the instance pool initially)

Click Next.

Click Create.

We have now created an autoscaling policy that will start with creating 1 compute instance in the designated pool. Once the CPU utilization is determined to be above 10% for at least 300 seconds, another compute instance will be launched automatically. Once the CPU utilization is determined to be less than 5% for 300 seconds, one compute instance will be removed. At all times, there will be at least 1 compute instance in the pool.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/16.JPG)

## 6) Test the AutoScaling setup

Go to Compute -> Instance Pool page. Check the created Instance pool in the list. The state could be in Scaling. Wait until your Instance Pool change from Scaling to Running state. After the state change click the Instance pool, in that page Click Created Instances, you should see a compute instance created. Click the Compute Instance name.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/16.1.png)

Note down the Public and Private IP of compute instance from the details page (Under Primary VNIC Information section).

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/16.2.png)

Open a web browser and enter instance's public IP address. You should see the message: Web Server IP: <instance private IP>

Login into the server via Putty. You need to specify private key while making the connection.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/17.JPG)

We are going to introduce CPU load manually through stress command. Check whether the stress command library is installed in the compute machine. Our init script has command to install stress via yum. Sometimes yum repository might not have library for stress.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/20.JPG)

In that case you can manually transfer rpm file through sftp 

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/22.JPG)

Then install it manually through yum localinstall <stress.rpm file>

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/23.JPG)

Now start CPU stress, Enter command:

        sudo stress --cpu 4 --timeout 350

Spawn 4 workers spinning on sqrt() with a timeout of 350 seconds.

Switch back to OCI console and navigate to Instance Pool Details page. Click your instance name and scroll down to Metric screen, you should see CPU spiking up after a minute or so.

![Compute instance VM](https://github.com/kmkittu/AutoScaling/blob/main/24.JPG)


Navigate to your Instance Pool details page. In about 3-4 minutes (time configured when we created auto scale configuration), status of Pool should change to Scaling and a second compute instance should launch.

This is since our criteria of CPU utilization > 10 was met.

When the second instance is up and running and the instance pool status is 'Running', switch to the web browser and refresh the page multiple times and observe the load balancer balancing traffic between the two web servers.

Switch back to git bash window and if the stress tool is still running, click Ctrl + C to stop the script. Switch back to OCI console window and navigate to your compute instance details page. Verify CPU utilization goes down after a minute. 
Navigate to Instance pool details page and after 3-4 minute Instance pool status will change to Scaling . Additional compute instance will be deleted.
sThis is because our criteria of CPU utilization < 5 is met.