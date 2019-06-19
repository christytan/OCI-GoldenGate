<table class="tbl-heading"><tr><td class="td-logo">![](images/obe_tag.png)

June 13, 2019
</td>
<td class="td-banner">
# Lab 7: Building and deploying Python application stacks on dedicated autonomous infrastructure
</td></tr><table>

## Introduction


This lab walks through how to confugure a linux host machine running on Oracle Cloud Infrastructure to run your Python applications and connect to Oracle Autononous Transaction Processin Dedicated database.

You are provided with a sample python application for this lab.


To **log issues**, click [here](https://github.com/cloudsolutionhubs/autonomous-transaction-processing/issues/new) to go to the github oracle repository issue submission form.

## Objectives

As an application developer,
- Learn how to deploy a python application and connect it to your dedicated autonomous database instance

## Required Artifacts

-  The following lab requires an Oracle Public Cloud account. You may use your own cloud account, a cloud account that you obtained through a trial, or a training account whose details were given to you by an Oracle instructor.

- This lab requires Oracle Cloud Developer Image launched in Compute instances.

## Steps

### **STEP 1: SSH into Oracle Cloud Developer image and clone Java application**

- Login to your Oracle Cloud Infrastructure account and click on **Menu** and select **Compute** and **Instances**

![](./images/800/Compute1.png)

- Select the right Oracle Developer Cloud image you created in earlier labs. 

- Copy the public IP address of the instance in a note pad. 

![](./images/800/Compute2.png)


### For Mac users

- Open Terminal and SSH into linux host machine

```
sudo ssh -i /path_to/sshkeys/id_rsa opc@publicIP
```

![](./images/800/SSH1.png)

### For Windows users

- You can connect to and manage linux host mahine using SSH client. Recent versions of Windows 10 provide OpenSSH client commands to create and manage SSH keys and make SSH connections from a command prompt.

- Other common Windows SSH clients you can install locally is PuTTY. Click [here](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ssh-from-windows) to follow the steps to connect to linux host machine from you windows using PuTTY.

### Cloning Python Application

- Once you have successfully SSH into the linux host machine create a new directory in /home/opc/

```
cd /home/opc/

mkdir ATPDpython
```


- Clone ATPDpython git repository into ATPDpython directory 

```
cd /home/opc/ATPDpython/

git clone https://github.com/dannymartin/ATPDpython.git
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
sudo scp -i /Path/to/your/private_ssh_key /Path/to/your/downloaded_wallet opc@publicIP:/home/opc/
```
![](./images/800/atpd5.png)

- You are now ready to move to step 3

### **STEP 3: Confuigure env variables and run python application in linux host machine**

We have now succesfully secured copied wallet into our linux host machine. We will now configure env variuables and database config files and run our python applicaiton.

- Open terminal in your laptop and SSH into linux host machine

```
ssh -i /path/to/your/private_ssh_key opc@PublicIP
```

- Create a new directory for wallet and unzip the wallet

```
cd /home/opc/ATPDpython/

mkdir wallet

unzip Wallet_ATPDedicatedDB.zip -d /home/opc/ATPDpython/wallet/
```

- Configure sqlnet.ora in our wallet folder

```
vi /home/opc/ATPDpython/wallet/sqlnet.ora
```

- Change **DIRECTORY** path to /home/opc/ATPDpython/wallet/ and save the file

![](./images/700/walletPython.png)

- Export TNS_ADMIN

```
export TNS_ADMIN=/home/opc/ATPDpython/wallet/
```

- Verify TNS_ADMIN path

```
echo $TNS_ADMIN
```
![](./images/700/TNSadmin.png)

- Run the python program to connect to the ATP database

```
 python exampleConnection.py ADMIN PASSWORD dbname_tp
```
![](./images/700/pythonSuccess.png)


-   You are now ready to move on to the next lab.


<table>
<tr><td class="td-logo">[![](images/obe_tag.png)](#)</td>
<td class="td-banner">

</td>
</tr>
<table>