# Slurm for Dummies 
A step-by-step guide on how to setup Slurm HPC clusters written for dummies by dummies from the UIowa Quant Finance Club 2023. We are by no means experts, but what is enclosed herein was learned through grueling trial and error.

### Table of Contents
- [Overview](#step-by-step-overview)
- [Set up SSH on each computer](#setting-up-ssh)
- [Setting up Munge](#setting-up-munge)
- [Setting up Control Node](#setting-up-control-node)
- [Setting up Worker Nodes](#setting-up-worker-nodes)
- [Other Resources](#other-resources)
- [FAQ](#faq)

## Step-by-Step Overview
These are the steps we followed to setup our Slurm cluster. It is important that you follow the steps in the sequence as they are written. Again, this is just what worked for us on fresh installs of Ubuntu 22.04LTS. 
> IMPORTANT: Steps marked with __(CONTROL NODE)__ are just performed on you control node and steps marked with __(WORKERS)__ are just performed in your worker nodes. Steps that aren't marked are performed in both.

1. Install Ubuntu 22.04 on all computers in the cluster.
2. Make sure all users on each computer have the same name. We will call this user on each computer MAIN_USER from here on out.
3. Make sure that all computers on the cluster have each other in their known hosts file. This file can be found at `/etc/hosts`. To add a known host to the file, you have to add the hosts IP address and the hosts alias separated by a space on a newline in the file. Our `/etc/hosts` file looked something like this:
``` 
127.0.0.1 localhost
XXX.XXX.XX.XX0	cluster0
XXX.XXX.XX.XX1	cluster1
XXX.XXX.XX.XX2	cluster2
XXX.XXX.XX.XX3	cluster3
XXX.XXX.XX.XX4	cluster4
```
> Note that the Xs here stand for numbers in our IP addresses. The aliases are also arbitrary, you can name your nodes whatever you like.

4. Run the following commands in your shell on each computer to update and upgrade all packages in that system.

```
$ sudo apt update
$ sudo apt upgrade
```
  
5. [Set up SSH on each computer](#setting-up-ssh)
6. __(CONTROL NODE)__ Set up [Munge](#setting-up-munge) on your control node first.
7. __(WORKERS)__ Set up [Munge](#setting-up-munge) on each of the worker nodes.
8. Setup [Slurm](#setting-up-slurm) on all machines. Make sure to follow the control node instructions for the control node and the worker node instructions for the worker nodes.

## Setting up SSH
Seting up SSH is pretty simple. You just run the following command:
```
$ sudo apt install openssh-server openssh-client
```
Then, to test whether it was installed correctly, we can attempt to SSH into another computer in the cluster like so:
```
$ ssh <hostname>			 (<hostname> would be the alias of another cluster. For example, cluster1 if you’re on cluster0) 
```
If SSH is succesful, you should know be in a remote shell connected to the host with name `<hostname>`. If you want to learn more about SSH, visit this [page](https://ubuntu.com/server/docs/service-openssh).

Remember to do this on each computer.

## Setting up Munge
### Control Node
__IN PROGRESS__
### Worker Nodes
__IN PROGRESS__
## Setting up Slurm
### Control Node
__IN PROGRESS__
### Worker Nodes
__IN PROGRESS__
## Other Resources
These are some resources we found helpful along the way. 
- [Munge docs](https://dun.github.io/munge/)
- [Slurm docs](https://slurm.schedmd.com/overview.html)
- [Blog on how to set up Slurm](https://www.bodunhu.com/blog/posts/set-up-slurm-across-multiple-machines/)

## FAQ
### What is Slurm?
Slurm is a cluster managament and job scheduling system for Linux clusters. It has very extensive documentation that can be found [here](https://slurm.schedmd.com/quickstart.html). 

### Why did we write this?
We are a group of students from the University of Iowa Quant Finance Club who struggled for weeks with setting up a Slurm cluster. We made every mistake in the book and looked everywhere for guides on how to setup clusters, but the guides we found were either going above our heads or missing critical information. So, we decided to document our process and put it on the web and, hopefully, it'll be able to help other dummies set up HPC clusters.

