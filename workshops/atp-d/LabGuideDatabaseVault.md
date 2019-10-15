<table class="tbl-heading"><tr><td class="td-logo">![](images/obe_tag.png)

June 13, 2019
</td>
<td class="td-banner">
# Lab 1: Preparing your private data center in the Oracle Cloud Infrastructure 
</td></tr><table>

To **log issues**, click [here](https://github.com/oracle/learning-library/issues/new) to go to the github oracle repository issue submission form.

## Introduction

Oracle Autonomous Database supports Oracle Database Vault, which you can use to implement powerful security controls within your dedicated database. These unique security controls restrict access to application data by privileged database users, reducing the risk of insider and outside threats and addressing common compliance requirements.

You can deploy controls to block privileged account access to application data and control sensitive operations inside the database. Trusted paths can be used to add additional security controls to authorized data access and database changes. Through the runtime analysis of privileges and roles, you can increase the security of existing applications by implementing least privileges and reducing the attack profile of your database accounts. Oracle Database Vault secures existing database environments transparently, eliminating costly and time consuming application changes.

## Objectives

To configure and enable Enable Oracle Database Vault in your dedicated Autonomous Database


## Required Artifacts

-   The following lab requires an Oracle Cloud account. You may use your own cloud account, a cloud account that you obtained through a trial, or a training account whose details were given to you by an Oracle instructor.
- Access to OCI Autonomous transaction processing console
- A pre-provisioned autonomous database with admin access
- A pre-provisioned developer client with network access to database

## Steps

### **STEP 1: Create Database Vault Owner, Account Manager, and MYAPP Schema**

-	Connect as the ADMIN user to your dedicated database.

-	Create the Database Vault owner and account manager users; for example:

```

create user dbv_owner identified by WElcome_123#;
grant create session to dbv_owner;
create user dbv_acctmgr identified by WElcome_123#;
grant create session to dbv_acctmgr;

```

![](./images/1200/createUsers.png)

-	The MYAPP schema will act as a schema that we **DO NOT** want the ADMIN or DBA to be able to access. Create the MYAPP schema and let's create a table Test within the schema that we will use later

```
create user myapp identified by WELcome_123#;
grant create session to myapp;
grant create table to myapp;
grant unlimited tablespace to myapp;
```

![](./images/1200/myapp.png)

![](./images/1200/createSessionmyapp.png)

-	**Connect as the MYAPP user** and let's create a table in this schema and add a couple rows for testing.

```
create table test(column1 int, column2 int);
insert into test values(1,2);
insert into test values(3,4);
commit;
```


-	Next, let's verify that Database Vault is not turned on with the command:

```
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Oracle Database Vault';
```

![](./images/1200/valueFalse.png)


### **STEP 2: Configure Database Vault**

-	Login as the ADMIN user. Provide the owner and account manager user names that we created in Step 1

```

exec dvsys.configure_dv('dbv_owner','dbv_acctmgr');

```

![](./images/1200/configure.png)


### **STEP 3: Create Realm and Enable Database Vault**

-	Connect as the Database Vault owner (dbv_owner)

-	Let's create a Realm that will prevent the ADMIN user or DBA's from accessing 

```
exec DBMS_MACADM.CREATE_REALM('atpd_realm','description','y',DBMS_MACUTL.G_REALM_AUDIT_OFF,0,1,FALSE); 
```

![](./images/1200/createRealm.png)

-	Now let's add the MYAPP table test to the Realm

```
exec DBMS_MACADM.ADD_OBJECT_TO_REALM('atpd_realm', 'myapp', 'test', 'TABLE'); 
```

![](./images/1200/addObjectToRealm.png)


-	Next, let's enable the Database Vault

```

exec dbms_macadm.enable_dv;

```

![](./images/1200/enable.png)


- Restart the Autonomous Database instance

- Once the Database is back up and running, let's verify that Database Vault is turned on.

- Connect as ADMIN user and run:

```
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Oracle Database Vault';
```

![](./images/1200/enable.png)

- We can see that now the value is TRUE. We have enabled Database Vault

- Even though we are the ADMIN user we should NOT be able to access MYAPP's table test because we have added it to the realm atpd_realm


- Connect as ADMIN user and attempt to query the MYAPP test table

```
select * from myapp.test;
```

![](./images/1200/insufficientPriv.png)


- We have successfully enabled Database Vault


<table>
<tr><td class="td-logo">[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
### You have learned a few tips and tricks
</td>
</tr>
<table>
