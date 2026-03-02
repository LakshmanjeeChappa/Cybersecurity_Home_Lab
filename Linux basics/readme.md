Chapter 2 – Linux Security Basics
Overview

In this lab, I performed core Linux system administration and security tasks using an Ubuntu Virtual Machine. The objective was to understand system updates, privilege management, user and group administration, file permissions, and Access Control Lists (ACLs). All commands were executed through the terminal.

1. Retrieve Available Updates
Command
sudo apt update
Explanation

The apt update command refreshes the local package index by retrieving the latest metadata from configured repositories listed in /etc/apt/sources.list. This command does not install updates; it only checks for available newer versions of installed packages.

Running this command ensures the system is aware of available security patches and software updates.



2. Upgrade the System
Command
sudo apt upgrade
Explanation

The apt upgrade command installs the newer versions of packages identified during apt update. This applies security patches and feature updates.

Upgrading the system reduces exposure to known vulnerabilities and ensures the operating system is running patched software.

Screenshot
![alt text](image.png)
![alt text](image-1.png)
3. Reboot the System
Command
sudo reboot

After reboot:

uname -r
Explanation

After upgrading, a reboot was required because the Linux kernel was updated. The uname -r command confirms the currently running kernel version. Rebooting ensures that the new kernel is loaded into memory.

Screenshot
![alt text](image-2.png)
4. Switch to Root User
Command
sudo su root
Explanation

This command switches the current user to root. The prompt changes from $ to #, indicating administrative privileges. The root user has unrestricted access to system files and configurations.

Screenshot
![alt text](image-3.png)
5. Create Users Using useradd and adduser
Commands
useradd bobby
adduser sally
![alt text](image-4.png)

![alt text](image-5.png)
Explanation

useradd is a low-level command that creates a user account without creating a home directory or setting a password by default.

adduser is a higher-level interactive tool that:

Creates a home directory

Sets a password

Copies default configuration files from /etc/skel

Creates a matching group

This demonstrates the difference between basic account creation and fully configured user setup.

6. Switch to Sally
Command
su - sally
![alt text](image-6.png)
Explanation

This command switches the session to user sally and loads her login environment. The prompt changes to $, indicating standard user privileges.

7. Attempt to Create a User as Sally
Command
useradd earl
![alt text](image-7.png)
Result

Permission denied.

Explanation

Creating a user requires modification of /etc/passwd and /etc/shadow, which are restricted system files. Sally does not have administrative privileges, so Linux denied access. This demonstrates privilege enforcement and the principle of least privilege.

8. Delete User Bobby
Command
sudo userdel bobby
![alt text](image-8.png)
Explanation

This removed the bobby user from the system account database.

9. Change Sally’s Password
Command
sudo passwd sally
![alt text](image-9.png)
Explanation

This command updated Sally’s password hash in /etc/shadow. Only root or sudo users can modify passwords for other users.

10. Why It Is Bad Practice to Stay Logged in as Root

Remaining logged in as root increases security risk. Root has unrestricted control over system files, users, and services. Accidental mistakes can damage the operating system. Additionally, if malicious software runs while logged in as root, it gains full system access. Using sudo for temporary privilege escalation follows the principle of least privilege and improves system security.

11. View User ID
Command
id
![alt text](image-10.png)
Explanation

The id command displays:

UID (User ID)

GID (Group ID)

Group memberships

Root always has UID 0. Regular users typically start at UID 1000.

12–16. Group Management
View Groups
groups lakshman
id sally
Create New Group
sudo groupadd cybersec
Add Sally to Group
sudo usermod -aG cybersec sally
![alt text](image-11.png)

![alt text](image-12.png)

![alt text](image-13.png)

![alt text](image-14.png)

![alt text](image-15.png)

![alt text](image-16.png)
Explanation

Groups allow role-based access control. Adding a user to a group grants access permissions assigned to that group. The -aG option appends the group without removing existing memberships.

17. Directory Permissions
Command
mkdir lab1
ls -ld lab1
![alt text](image-17.png)
Output Example
drwxrwxr-x
Explanation

Permissions are divided into:

Owner

Group

Others

rwx means read, write, execute.

For directories:

Read = list contents

Write = create/delete files

Execute = enter directory

18. Create Bash Script
Script Contents
#!/bin/bash
echo "Hello World!"
Commands
chmod +x helloWorld
./helloWorld

![alt text](image-18.png)
Explanation

The shebang (#!/bin/bash) tells the system which interpreter to use. chmod +x adds execute permission, allowing the file to run as a program.

19. File Permissions
Command
ls -l helloWorld
Example Output
-rwxrwxr-x
![alt text](image-19.png)
This shows:

Owner: full permissions

Group: full permissions

Others: read and execute

20. View Access Control List (ACL)
Command
getfacl helloWorld
![alt text](image-20.png)
Explanation

ACLs provide more granular permission control beyond standard UNIX permission bits. They allow specific users or groups to be granted customized access.

21. Modify ACL
Command
setfacl -m u:sally:rw helloWorld
![alt text](image-21.png)
Explanation

This grants user sally read and write access to the file, even if she is not the owner. The mask field limits the maximum effective permissions for named users and groups.