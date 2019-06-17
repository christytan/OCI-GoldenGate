<table class="tbl-heading"><tr><td class="td-logo">![](images/obe_tag.png)

June 13, 2019
</td>
<td class="td-banner">
# Lab 6: Building and deploying node.js application stacks on dedicated autonomous infrastructure
</td></tr><table>

## Introduction

To **log issues**, click [here](https://github.com/cloudsolutionhubs/autonomous-transaction-processing/issues/new) to go to the github oracle repository issue submission form.

## Objectives

- Learn how to build a linux Node application and connect it to an Oracle ATP Dedicated database

## Required Artifacts

-  The following lab requires an Oracle Public Cloud account. You may use your own cloud account, a cloud account that you obtained through a trial, or a training account whose details were given to you by an Oracle instructor.

- This lab requires Oracle Cloud Developer Image launched in Compute instances.

## Steps

### **STEP 1: SSH into Oracle Cloud Developer image and clone Node application**

- Login to your Oracle Cloud Infrastructure account and click on **Menu** and select **Compute** and **Instances**

![](./images/800/Compute1.png)

- Select the right Oracle Developer Cloud image you created in earlier labs. 

- Copy the public IP address of the instance in a note pad. 

![](./images/800/Compute2.png)


### For Mac users

- Open Terminal and SSH into linux host machine, we will need port forwarding to see the Node application running in our local browser. If you use VNC then there is no need to configure port forwarding

```
sudo ssh -i /path_to/sshkeys/id_rsa -L 3050:127.0.0.1:3050 opc@publicIP
```

![](./images/800/SSH1.png)

### For Windows users

- You can connect to and manage linux host mahine using SSH client. Recent versions of Windows 10 provide OpenSSH client commands to create and manage SSH keys and make SSH connections from a command prompt.

- Other common Windows SSH clients you can install locally is PuTTY. Click [here](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ssh-from-windows) to follow the steps to connect to linux host machine from you windows using PuTTY.

### Cloning Node Application

- Once you have successfully SSH into the linux host machine create a new directory in /home/opc/

```
cd /home/opc/

mkdir ATPDnode
```


- Clone ATPDnode git repository into ATPDnode directory 

```
cd /home/opc/ATPDnode/

git clone https://github.com/dannymartin/ATPDnode.git
```

- You are now ready to move to step 2

### **STEP 2: Secure Copy ATP Dedicated database wallet to linux host machine**

- Login to Oracle Cloud Infrastructure account and click on **Menu** and **Autonomous Transaction Processing**
![](./images/800/atpd1.png)

- Click on Autonomous Database and select your previously created database

![](./images/800/atpd2.png)

- Click on DB Connection and under Download Client Credential(Wallet) click **Download**

![](./images/800/atpd3.png)

- Database connections to you Autonomous Database use a secure connection. You will be asked to create a password for yopu wallet. 

- Enter **Password** and **Confirm password** and click on **Download**

![](./images/800/atpd4.png)

- The credentials zip file contains the encryption wallet, Java keystore and other relevant files to make a secure TLS 1.2 connection to your database from client applications. Store this file in a secure location.

- Let us now secure copy the downloaded wallet to our linux host machine

- Open Terminal in your laptop and type in the following commands

#### Note: Please change the path for both private ssh key and wallet in below command

```
scp -i /Path/to/your/private_ssh_key /Path/to/your/downloaded_wallet opc@publicIP:/home/opc/
```
![](./images/800/atpd5.png)

- You are now ready to move to step 3

### **STEP 3: Confuigure env variables and run node application in linux host machine**

We have now succesfully secured copied wallet into our linux host machine. We will now configure env variuables and database config files and run our node applicaiton.

- Open terminal in your laptop and SSH into linux host machine

```
ssh -i /path/to/your/private_ssh_key opc@PublicIP
```

- Create a new directory for wallet and unzip the wallet

```
cd /home/opc/ATPDnode/

mkdir wallet

unzip Wallet_ATPDedicatedDB.zip -d /home/opc/ATPDnode/wallet/
```

- Configure sqlnet.ora in our wallet folder

```
vi /home/opc/ATPDnode/wallet/sqlnet.ora
```

- Change **DIRECTORY** path to /home/opc/ATPDnode/wallet/ and save the file

![](./images/700/walletNode.png)

- Export TNS_ADMIN

```
export TNS_ADMIN=/home/opc/ATPDnode/wallet/
```

- Verify TNS_ADMIN path

```
echo $TNS_ADMIN
```
![](./images/700/TNSnode.png)

- Run the node application

```
node server.js 
```
![](./images/700/runNode.png)

- Open a browser and go to localhost:3050 to see the successful connection!

![](./images/700/connectionSuccessful.png)


- You are now ready to move to the next lab.

<table>
<tr><td class="td-logo">[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
## Great Work - All Done!
</td>
</tr>
<table>