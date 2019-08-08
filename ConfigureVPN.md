<table class="tbl-heading"><tr><td class="td-logo">![](./images/obe_tag.png)

</td>
<td class="td-banner">
# Lab 5: Configuring VPN connectivity into your private ATP network
</td></tr><table>


## Introduction

Oracle's dedicated Autonomous Infrastructure and databases are deployed in a private VCN in the Oracle Cloud Infrastructure with no public IP address assigned. Hence to gain connectivity to the databases, a best practice approach is to use a VPN connection. 

This lab walks you through the steps to deploy a VPN server in OCI and create an SSL VPN connection between a client machine (your desktop) and the dedicated ATP infrastructure. Once configured, a single VPN server can be shared among multiple users.

To **log issues**, click [here](https://github.com/oracle/learning-library/issues/new) to go to the github Oracle repository issue submission form.

## Objectives
As a network or fleet admin,

- Configure a VPN server in OCI based on OpenVPN software
- Configure your VPN client and connect to VPN Server
- Launch SQL Developer on client and connect to a dedicated ATP instance



## Required Artifacts

- An Oracle Cloud Infrastructure account with privileges to create compute instance and network resources
- A pre-provisioned dedicated autonomous database instance in a private network
- A pre-provisioned Virtual Cloud Network with public and private subnets setup with appropriate security lists. Refer [Lab 1](./ATPD-Networking.md)


The following illustration shows a network topology that can be used to provide secure access to your dedicated autonomous infrastructure.
![](./images/1200/highlevelSSL.png)

- As shown above, You OCI Virtual Cloud Network (VCN) has two subnets. A private subnet with CIDR 10.0.0.0/24 that hosts your exadata infrastructure and a public subnet with CIDR 10.0.1.0/24 that has public facing web and application servers and also the VPN Server. 

- An internet gateway is attached to the public subnet to allow all resources within that subnet to be accessible over the internet.

- Security lists have been setup such that tcp traffic into the private exadata subnet is allowed only through hosts in the public subnet. This can be further tightened by allowing traffic from specific hosts and ports. 

- For detailed instructions on network setup for your dedicated autonomous DB infrastructure, refer to [Lab 1](ATPD-Networking.md)




## Steps


### **STEP 1: Launch a CentOS VM for the OpenVPN server**

- Login to the Oracle Cloud Infrastructure using your tenancy, userId and password. 

    Refer to Lab 1 for detailed instructions on logging into your OCI account.

- Once logged in, Click on **Create Instance**

![](./images/1200/createCompute.png)




- Name your instance and select **CentOS7** as your image source 

![](./images/1200/ComputeImage.png)

- Select **Virtual Machine** and add your public SSH key file 

![](./images/1200/ComputeType.png)

-  Next, select the network for your VPN Server
    - Select the compartment & VCN where your exadata infrastructure is provisioned
    - Select the compartment where your public subnet is provisioned
    - Pick public subnet from the drop down
 
    
![](./images/1200/ComputeNetwork.png)

#### Note that while your ATP infrastructure and VPN server are in the same VCN, ATP is in a private subnet while the VPN server is deployed in a public subnet for access over the internet. 




-  Click Create and within a few mins your CentOS server will be ready with a public IP for ssh access



### **STEP 2: Install and Configure OpenVPN Server**

-   ssh into centOS vm and download the openVPN rpm package

    ```
    $ ssh opc@O<public_ipAddress_of_your_centOS_VM>
    ```
    ```
    $ wget http://swupdate.openvpn.org/as/openvpn-as-2.5.2-CentOS7.x86_64.rpm
    ```

   
![](./images/1200/openvpn_configure.jpeg)


-   Use the RPM command to install the package

        $ sudo rpm -ivh openvpn-as-2.5.2-CentOS7.x86_64.rpm
![](./images/1200/openvpn_url.jpeg)

-   Change password of OpenVPN Server

    ```
    $ sudo passwd openvpn
    ```

-    From your local browser, access the admin UI console of your VPN Server (**https://public_ipAddress_of_your_centOS_VM:943/admin**), using the password for OpenVPN user
 

![](./images/1200/openvpn_login.png)

-   Once you are logged in, click **Network Settings** and replace the **Hostname or IP address** with the public IP of the OpenVPN Server Instance

![](./images/1200/openvpn_network.png)

**Save your setting before advancing to the VPN settings page**

-   Click **VPN settings** and add CIDR ranges for both the app Subnet and the exadata Subnet. The exadata subnet CIDR can be avoided if you DO NOT plan to connect directly to your databases using a desktop SQL Client like SQL Developer.
Note that in the **Routing** section, ensure that the option **Should client Internet traffic be routed through the VPN?** is set to **YES**

![](./images/1200/openvpn_vpnsetting.png)


-   Under **Have Clients user these DNS servers**, pick a pair of public DNS resolvers. Here we choose Google's DNS resolvers 8.8.8.8 and 8.8.8.4

![](./images/1200/openvpn_DNS.png)

**Save your setting before advancing to the Advanced VPN settings page**

-   In the **Advanced VPN** section, ensure that the option **Should clients be able to communicate with each other on the VPN IP Network?** is set to **Yes**

![](./images/1200/openvpn_advancedVPN.png)

Note: Once you have applied your changes, click **Save Settings** once again. Then, **Update Running Server** to push your new configuration to the OpenVPN server.


### **STEP 3: Install OpenVPN Client**

-   Launch your OpenVPN Access Server Client UI at **https://Your_VPN_Server_Public_IP:943** and download the OpenVPN client for your platforms.
    
    ![](./images/1200/openvpn_client.png)
    

-   Once the installation process has completed, you can see an OpenVPN icon in your OS taskbar. Right-Click this icon to bring up the context menu to start your OpenVPN connection

    ![](./images/1200/openvpn_conn.png)
    ![](./images/1200/openvpn_client_conn.png)
    
    ##### Note: IP should be Public IP for OpenVPN Compute Instance

-   Click **Connect** brings up a window asking for the OpenVPN username and password. Enter the credentials for your **openvpn** user and click **Connect** to establish a VPN tunnel

    ![](./images/1200/openvpn_clientwindow.png)


You may also setup your VPN server with multiple users. Follow the OpenVPN configuration guide to setup additional users.


### **STEP 4: Configure and connect SQL Developer to your dedicated ATP database**

Before you launch and connect SQL Developer, you have to ensure that the SCAN host name in tnsnames.ora can resolve to an IP address for your exadata infrastructure. Since your exadata infrastructure is in a private network, it has a private ip from the 10.0.0.0/24 CIDR pool and is therefore not resolvable by public DNS servers. 

You need to, 

a. Find the private IP address if your dedicated autonomous database host which in this case is something like 10.0.0.X

b. Edit your local host file and add an entry for the SCAN host

Once you do you this, you can them simply launch SQL Developer and use the database wallet to make a connection to your ATP-D instance as long as you are on VPN




#### Lets first find the SCAN host name from the credentials wallet file.

- Download the wallet .zip file to your local machine
- Unzip file and open tnsnames.ora in an editor
- The SCAN host name appears similar to the one shown below. Copy the SCAN host name some place. 

    ![](./images/1200/atpd_hostname.png)
    

    
Since your autonomous exadata infrastructure is setup as a RAC configuration, the SCAN host resolves to 3 IP addresses to load balance between the RAC nodes. You may use any one of those IP addresses for connectivity

#### Lets find the private IP address of your exadata RAC nodes
    
- ssh into your VPN server. You may use public or private IP. If you are on VPN, the private IP should work.

    ```
    $ ssh opc@<IP_address_of_VPN_Server>
    ```
 Query OCI DNS for private IP of Exadata SCAN Host

    
        $ nslookup SCAN_Hostname_from_tnsnames.ora

    
![](./images/1200/atpd_privateIP.png)
    
    
Now that you have an IP address for your autonomous exadata infrastructure, update the hosts file on your local machine (/etc/hosts on a mac or the C:\Windows\System32\Drivers\etc\hosts on Windows) 
-   Add ATPD Private IP and Hostname in Local Platform

    e.g. on a Mac: add IP and hostname to /etc/hosts

    

    ```
    10.x.x.6 hosxxxxxxxan.exadatasubnet.ocivcn.oraclevcn.com
    ```

-   You are now ready to connect to your dedicated ATP instance using a local SQL Client such as SQL Developer


Launch SQL Developer and connect using the downloaded credentials wallet as shown below

![](./images/1200/atpd_conn.png)
    
Follow detailed instructions on downloading your database credentials wallet refer to [Lab 4](ProvisionADB.md) 

You may also connect to APEX or SQL Developer Web directly from your local browser. Simply get the URL from the console and launch in a browser window
   

![](./images/1200/atpd_application_apex.png)
        


<table>
<tr><td class="td-logo">[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
## Congratulations! You just configured a secure VPN connection into your private autonomous exadata infrastructure.
</td>
</tr>
<table>
