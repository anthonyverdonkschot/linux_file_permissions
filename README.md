# Linux File Permissions

## Project scenario

You are a security professional who works with the research team at a large organization. You’re responsible for ensuring that users on the research team are authorized with appropriate permissions. Your task is to evaluate current permissions on the file system and modify permissions based on authorization instructions.

## Check file and directory details

To check file and directory permissions in Linux, use the `ls` command with the `-l` option. To also view hidden files, use the `-a` option.

These two options can be combined: `ls -la`

![ls -la](/images/LFP_1.png)

The group of letters on the left side of the output is known as the permissions string.

## Describe the permissions string

The permissions string consists of 10 characters. 

- The first character indicates if you’re dealing with a file (`-`) or directory (`d`).
- The following 9 characters are divided into three sections: user, group, and other. Characters within these sections include `r` for read, `w` for write, `x` for execute, and `-` for none.
    
<img src="/images/LFP_2.png" width="800" alt="Permission diagram for project_k.txt" />
    
For example, let’s take a look at the permissions string for `project_k.txt`.

![-rw-rw-rw-](/images/LFP_3.png) 

The first character (`-`) indicates that it is a file. The permissions for user, group, and other are allowed read and write access but not execute access (`rw-rw-rw-`).

## Change file permissions

According to the organization, other should not be allowed write access to any files. 
Currently, `project_k.text` allows write access for other.

![-rw-rw-r(w)-](/images/LFP_4.png)

**chmod**

The `chmod` command is used in Linux to modify file and directory permissions. With `chmod` in symbolic mode, you first include one of three letters: `u` for user, `g` for group, or `o` for other. Next you can use `+` to add, `-` to remove, or `=` to assign the permissions: `r`, `w`, or `x`. Lastly, write the name of the file or directory that you intend to change permissions for.

To remove write (`w`) permissions from other (`o`)  use the following command:

![chmod o-w project_k.txt](/images/LFP_5.png)

Updated permissions for `project_k.txt`

![-rw-rw-r--](/images/LFP_6.png)

## Change file permissions on a hidden file

The hidden file `.project_x.text` was recently archived and should not allow write access for anyone. Only the user and group should have read access.

> Remember to include the `-a` flag in the `ls -la` command to view hidden files.

Current permissions for `.project_x.txt` indicate that the user and group have write access:

![-rw--w----](/images/LFP_7.png)

The `chmod` command allows combinations of permission assignments with a comma (`,`). The following command demonstrates this by removing write access from the user and group (`u-w,g-w`) and then adding read access to group (`w+r`).

![chmod u-w,g-w+r .project_x.txt](/images/LFP_8.png)

Updated permissions for `project_x.txt`

![-r--r-----](/images/LFP_9.png)

## Change directory permissions

The organization decided that only the `researcher2` user should be allowed access to the `drafts` directory. No one else other than `researcher2` should have execute permissions.

Current permissions for drafts indicate that the group has execute permissions:

![drwx--x---](/images/LFP_10.png)

To remove group execute access to the `drafts` directory, use the following command:

![chmod g-x drafts](/images/LFP_11.png)
![drwx------](/images/LFP_12.png)

## Summary

Permissions were successfully identified and modified in Linux based on authorization instructions from the security organization. The `ls -la` command permitted the viewing of permissions for files and directories, including those hidden. The `chmod` command allowed for multiple permission modifications.

# Extra: Recreating the lab file structure

## Overview of the file structure

This project was based on a Linux lab from Google's Cybersecurity course. I decided to recreate the file structure from the lab onto my Kali Linux virtual machine (VM). 

**Here's a walkthrough of my process:**

To start, I created a backup snapshot of my Kali VM, so that I could revert changes made during the project.

![Backing up Kali Linux VM in Oracle VM VirtualBox Manager](/images/LFP_13.png)

The file structure of the lab consists of a user, `researcher2`, who has a directory called `projects` with the following files: `.project_x.txt`, `project_k.txt`, `project_m.txt`, `project_r.txt`, `project_t.txt`, and a directory named `drafts`.

`researcher2` is also in the `research_team` group.

<br>

The following diagram outlines the file structure:

<img src="/images/LFP_14.png" width="800" alt="Diagram of the file structure" />

### Creating the group and user

I created the `research_team` group using the `groupadd` command:

![groupadd research_team](/images/LFP_15.png)

To create the `researcher2` user, I used the `useradd` command. The `-m` option creates a directory for the user in `/home`; the `-g` option specifies a group.

![useradd -m -g research_team researcher2](/images/LFP_16.png)

I used the `id` command to ensure that the user was created:

![id researcher2](/images/LFP_17.png)

Upon trying to switch to the researcher2 with the `su` command I ran into a problem. The prompt only displayed `$`.

![$](/images/LFP_18.png)

After some quick research, I discovered that I did not set a login shell for this account. To add a login shell, I used the `usermod` command with the `-s` flag indicating the path of the bash shell. After this, I could switch to the `researcher2` user successfully.

![usermod -s /bin/bash researcher2, su researcher2](/images/LFP_19.png)

### Creating the projects directory and its contents

I created the `projects` directory with `mkdir`, used `ls` to ensure the directory creation, and used `cd` to move into the new directory:

![mkdir projects, ls, cd projects](/images/LFP_20.png)

Now, for the `.txt` files within `projects`, I thought about using the `touch` command to create them individually, but I was curious if there was a faster method. Upon research, I learned that the `{}` wildcard allows listing comma-separated values - known as brace expansion.

I implemented brace expansion to create the`project_k.txt`,`project_m.txt`,`project_r.txt`, and `project_t.txt` files in one command:

![touch project_{k,m,r,t}.txt](/images/LFP_21.png)

While researching the `{}` wildcard, I learned that a semicolon (`;`) separates multiple commands. I used semicolons to add two more commands to my line: one for creating the hidden `.project_x.txt` file and another for creating the `drafts` directory.

![touch project_{k,m,r,t}.txt; touch .project_txt; mkdir drafts, ls -a](/images/LFP_22.png)

Now that the file structure is in place, it’s time to set the permissions.

## Recreating the lab’s default file permissions

During the course, a provided document outlined the default file permissions of the lab environment. I created the following diagram to visualize the file and directory permissions of the lab.

<img src="/images/LFP_23.png" width="800" alt="Diagram of the permissions" />

Current permissions of the files and directories:

![ls -la](/images/LFP_24.png)

### Using chmod with the numeric method

For this task, I used the `chmod` command with the numeric method. The numeric method uses numbers to set all three owner classes at once. Here’s a quick summary of the values each number entails:

> Each write, read, and execute permissions have the following number value:
> 
> - `r` (read) = 4
> - `w` (write) = 2
> - `x` (execute) = 1
> - no permissions = 0
> 
> Credit: https://linuxize.com/post/chmod-command-in-linux/
> 

These numbers get added together with multiple permissions. For example, the value for having both read (4) and write (2) permissions would be 6 as (2 + 4 = 6).

The following diagram illustrates the `chmod` commands required to set the appropriate permissions:

<img src="/images/LFP_25.png" width="800" alt="Diagram of the permissions and chmod" />

Finally, I chained together each required `chmod` command to set all appropriate file and directory permissions.

![chmod 620 .project_x.txt; chmod 710 drafts; chmod 666 project_k.txt; chmod 640 project_m.txt; chmod 664 project_{r,t}.txt, ls -la](/images/LFP_26.png)

And with that, the permissions have been set to the lab defaults.
