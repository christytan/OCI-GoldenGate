<table class="tbl-heading"><tr><td class="td-logo">

![](./images/obe_tag.png)

Sept 1, 2019
</td>
<td class="td-banner">
# Install and configure a  workload generator
</td></tr><table>

To **log issues**, click [here](https://github.com/oracle/learning-library/issues/new) to go to the github oracle repository issue submission form.

## Introduction


**In this lab we will install and configure a workload generation tool called Swingbench to demonstrate online scaling, HA, Application Continuity and other cool automation features of the Oracle Autonomous database.**


## Objectives

As an adminstrator,
- Learn how to install and use Swingbench to simulate a transaction processing workload


## Required Artifacts

- An Oracle Cloud Infrastructure account

- A pre-provisioned instance of Oracle Developer Client image in an application subnet. Refer to [Lab2](20DeployImage.md)

- A pre-provisioned Autonomous Transaction Processing instance. Refer to [Lab 1](./10ProvisionAnATPDatabase.md)

## Steps

### **STEP 1: Log in to the Oracle Cloud Developer image and install Swingbench**

To connect to your Oracle Cloud Developer image please refer to [Lab2](20DeployImage.md). If  you are already connected from the previous lab skip to **STEP 2**.  

**The remainder of this lab assumes you are connected to the image through VNC Viewer and are operating from the image itself and not your local machine (except if noted)**

### STEP 2: Install and configure  Swingbench


ssh into your developer client machine following instructions in [Lab2](20DeployImage.md). Once you ssh  to the your developer client machine:

- Download swingbench into /home/opc using curl command below


````
curl http://www.dominicgiles.com/swingbench/swingbench261082.zip -o swingbench.zip
````

- unzip swingbench.zip. It unzips contents in the swingbench folder

```
unzip /home/opc/swingbench.zip
```

Let's also update the jdbc drivers in swingbench/lib to 18c drivers available at
https://www.oracle.com/database/technologies/appdev/jdbc-ucp-183-downloads.html

**Note: 19c drivers have not been tested with swingbench and may produce unpredictable results**

Download ojdbc8-full.tar.gz to the downloads folder in your dev. client instance

untar the archive

```
tar -xvf ojdbc10-full.tar.gz
```

Copy the contents of ojdbc10-full to /home/opc/swingbench/lib, replacing the existing files. 

Note: Do not modify the launcher directory or the existing swingbench.jar. 


### STEP 3: Transfer DB Wallet to swingbench client machine**

If you have not previously downloaded the wallet for your ATP database follow the steps below. If you previously downloaded the wallet, skip to **STEP 4**

- Create a VNC connection to your machine following steps in [Lab2](20DeployImage.md)
- Open up firefox in your VNC terminal sfrom the Applications menu on the top left.

 ![](./images/800/terminal.jpeg)

 -  Once Firefox is open go to **cloud.oracle.com**


- Login to Oracle Cloud Infrastructure account and select **Autonomous Transaction Processing** from menu
![](./images/800/atpd1.png)

- Click on Autonomous Database and select your previously created database

![](./images/800/atpd2.png)

- Click on DB Connection and under Download Client Credential(Wallet) click **Download**

![](./images/800/atpd3.png)

- Provide a password for your wallet and  download wallet

![](./images/800/atpd4.png)

- The credentials zip file contains the encryption wallet, Java keystore and other relevant files to make a secure TLS 1.2 connection to your database from client applications. The wallet will be downloaded to your downloads directory.

![](./images/800/walletdownload.jpeg)



### **STEP 4: Build and setup sample swingbench schema**

Now that you have downloaded the database wallet and installed Swingbench, the first step is to populate data in the database to use when creating the workload. From where we left of on  **STEP 2** above, change to the **bin** directory:

```
cd bin
```
So you should be in the **/home/opc/swingbench/bin** directory

```
./oewizard -cf ~/Downloads/your_wallet \
           -cs yourdbname_medium \
           -ts DATA \
           -dbap adminuserpassword \
           -dba admin \
           -u soe \
           -p adminuserpassword \
           -async_off \
           -scale 5 \
           -hashpart \
           -create \
           -cl \
           -v
```


![](./images/swingbench/loadsb.jpeg)

Understanding the parameters above 

- cf speficies the location of the wallet file
- cs specifies the ATP service to connect to 
- ts  name of the table space to install swingbench into. For ATP it is always DATA
- dba user admin created during ATP instance creation
- dbap admin user password
- u new database user created for swingbench data
- p new user (above) password 
- async_off async commits are not supported in  ATP
- scale in GB’s, data size, plus additional 50% of data for indexes 

Creation time will vary depending on where you are running it from and may take many minutes. The creation screen will look similar to this:

![](./images/swingbench/creating.jpeg)

Ignore any creation or priviledges error messages that display during the creation process. When finished, your screen should look similar to this:


![](./images/swingbench/creationfinish.jpeg)

- Once complete verify the tables created correctly by running the following command (from the swingbench bin directory, make sure to use your wallet location, and password):

```
./sbutil -soe -cf ~/Downloads/your_wallet.zip -cs yourdb_medium -u soe -p yourpassword -val
```

Your output should be similar to this:

![](./images/swingbench/oevalid.jpeg)

To collect statistics on the tables that were created run the command below. If you will be conducting performance testing with Swingbench it is recommended that you collect statistics.

```
./sbutil -soe -cf ~/Downloads/your_wallet.zip -cs yourdb_medium -u soe -p yourpassword -stats
```
To see how many rows were inserted on each table run the following command:

```
./sbutil -soe -cf ~/Downloads/your_wallet.zip -cs yourdb_medium -u soe -p yourpassword -tables
```

### STEP 4: Setup TAC parameters and run workload

There are atleast 2 options to run your swingbench workload. 

1. Using the Swingbench GUI. 

Simply fire up swingbench from a terminal window in your VNC session and select 'SOE_Client_Side' from the opening menu. This will use the Simple Order Entry workload using client side jdbc calls.

![](./images/swingbench/swingbench1.png)



You are ready to run Swingbench workloads on ATP. Workloads are simulated by users submitting transactions to the database. To do this, the user process must be configured. Run the following command unchanged from the same **bin** directory you have been running the other commands:

```
sed -i -e 's/<LogonGroupCount>1<\/LogonGroupCount>/<LogonGroupCount>5<\/LogonGroupCount>/' \
       -e 's/<LogonDelay>0<\/LogonDelay>/<LogonDelay>300<\/LogonDelay>/' \
       -e 's/<WaitTillAllLogon>true<\/WaitTillAllLogon>/<WaitTillAllLogon>false<\/WaitTillAllLogon>/' \
       ../configs/SOE_Server_Side_V2.xml
```

### **STEP 5: Generate workloads and scale ATP**

You can now generate loads on your database by running the charbench utility.  Use the command below. There are 2 parameters you can change to modify the amount of load and users being generated. ``The –uc flag specifies the number of users that will be ramped up, in the case below 64. The –rt flag specifies the total running time which is set to 30 seconds by default.``  You can stop running charbench at any time with **Ctrl C.**

```
./charbench -c /home/opc/swingbench/configs/SOE_Server_Side_V2.xml \
            -cf ~/Downloads/your_wallet.zip \
            -cs yourdb_tp \
            -u soe \
            -p yourpassword \
            -v users,tpm,tps,vresp \
            -intermin 0 \
            -intermax 0 \
            -min 0 \
            -max 0 \
            -uc 64 
```
Once swingbench starts running your will see results similar to the screen below. The first column is a time stamp, the second column indicates how many users of the total users requested with the **-uc** parameter are active, and of interest is the 3rd column which indicates transactions per second. If you see any intermittent connect or other error messages, ignore those.

![](./images/swingbench/swingbenchoutput.jpeg)



<table>
<tr><td class="td-logo">

[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
### Congratulations! You successfully completed setting up the swingbench workload generator for use in subsequent labs.




</td>
</tr>
<table>