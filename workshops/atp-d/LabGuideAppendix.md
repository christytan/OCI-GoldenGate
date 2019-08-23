<table class="tbl-heading"><tr><td class="td-logo">![](images/obe_tag.png)

June 13, 2019
</td>
<td class="td-banner">
# Lab 1: Preparing your private data center in the Oracle Cloud Infrastructure 
</td></tr><table>

To **log issues**, click [here](https://github.com/oracle/learning-library/issues/new) to go to the github oracle repository issue submission form.

## Introduction

The Oracle dedicated autonomous database runs on dedicated Exadata hardware in the Oracle Cloud Infrastructure. That means you have your own personal slice of high performance hardware akin to running your own private cloud in a public cloud setting. Lets take a look at some best practices to setting up your autonomous data platform.

In this section, we have a few tips and tricks for new Linux users

## Objectives

Learn Linux commands

### SSH

The ssh command is used to enable a secure connection to a remote machine. 

Example to connect to a remote machine using a private key
```
ssh -i privateKey user@123.456.78.89
```

### SCP

The scp command (secure copy) is used to transfer files between servers.

Example to move a local file to a remote server using a private key
```
scp -i privateKey /file/to/transfer.txt user@123.456.67.78:/directory/to/move/file/
```

### WGET

The wget command is used to retrieve files and information from a URL

Example to retrieve a webpage:

```
wget https://www.oracle.com
```

### MKDIR

The mkdir command is used to create new directories.

Example creating a directory:
```
mkdir newDirectory
```

### CD

The cd command is used to switch directories

Example changing directories if we are currently in /home/ and we want to move to /home/test/folder/
```
cd /home/test/folder
```


### ZIP and UNZIP

The zip command compresses files into a zip archive. The unzip command expands the zip archive.

Example compressing files and unzipping archives

```
zip archiveBeingCreated.zip files.txt added.txt abc.txt

unzip archiveBeingCreated.zip
```

### VI

The vi editor allows the user to edit and manipulate files from the command line

Once within the vi editor, "i", will enable interactive mode and allow the user to edit/add/delete lines.

Typing ":wq" will write/save the changes made and will then quit the vi editor.

To learn more, please visit this easy [tutorial](https://www.tutorialspoint.com/unix/unix-vi-editor.htm)


### EXPORT

The export command will set environment variables for the current shell process

Example exporting an environment variable:
```
export TNS_ADMIN=/home/wallet/
```

In the example above, we are setting the environment variable TNS_ADMIN in the shell process. The environment variable will no longer exist if we create a new session.


<table>
<tr><td class="td-logo">[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
### You have learned a few tips and tricks
</td>
</tr>
<table>
