# Slurm for Dummies 
A step-by-step guide on how to setup Slurm HPC clusters written for dummies by dummies from the UIowa Quant Finance Club 2023. We are by no means experts, but what is enclosed herein was learned through grueling trial and error.

### Table of Contents
- [Overview](#step-by-step-overview)
- [Set up SSH on each computer](#setting-up-ssh)
- [Setting up Munge](#setting-up-munge)
- [Setting up Slurm Service](#setting-up-slurm)
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
Installing Munge is pretty straightforward once you figure out what you're doing. However, the one thing that can get tricky is the file permissions, so make sure you follow the steps in order. Also, we recommend configuring the control node first and then configuring the worker nodes.

### Control Node
First, run the following command to install the munge packages.
```
$ sudo apt install munge libmunge2 libmunge-dev
```
This should install successfully as long as you're connected to the internet. To test your installation, you can run the following command:
```
$ munge -n | unmunge | grep STATUS
```
You should see something like `STATUS: SUCCESS`. Now, you have Munge correctly installed and there should be a Munge key at `/etc/munge/munge.key'. If you don't see one, then you should be able to create one manually by running the following command:
```
$ sudo /usr/sbin/mungekey
```
Now, we have to ensure all of the munge files have the correct permissions. This just entails giving the munge user ownership over all the munge files. You don't have to create the munge user manually since it should have been created by munge when we installed the packages above. In fact, we recommend saving yourself the trouble and not creating the user yourself. We had a lot of troubles stem from trying to create it ourselves.

To set up the correct permissions, use the following commands:
```
$ sudo chown -R munge: /etc/munge/ /var/log/munge/ /var/lib/munge/ /run/munge/
$ sudo chmod 0700 /etc/munge/ /var/log/munge/ /var/lib/munge/
$ sudo chmod 0755 /run/munge/
```

Next, we need to restart the munge service and configure it to run at startup. We do that like so:
```
$ sudo systemctl enable munge
$ sudo systemctl start munge (If you get an error here, try doing restart instead of start)
```
That's it! Now, you can go ahead and set up your worker nodes. Also, for convenience you can now save your `munge.key` located at `/etc/munge/' to an easily accessible location. You will need to copy that key over to the other nodes in the cluster when setting them up. We go over that in detail next.

### Worker Nodes
For each worker node we follow the same procedure. Similar to the control node, you first install munge, like so:
```
$ sudo apt install munge libmunge2 libmunge-dev
```
We check if munge is installed correctly, like so:
```
$ munge -n | unmunge | grep STATUS
```
Again, you should see something like `STATUS: SUCCESS`. Now, munge is correctly installed on this node, however we still need to copy our control node's key to this node. To do that, simply replace the worker node's `munge.key` file located at `/etc/munge/' with the control node's `munge.key` file. The most straightforward way we found to do this was to put the control node's 'munge.key' onto a USB drive and then plug the USB drive into the worker node. 

Once you have swapped out `munge.key`, we need to make sure munge's permissions are correct on this worker node. We do that like so:
```
$ sudo chown -R munge: /etc/munge/ /var/log/munge/ /var/lib/munge/ /run/munge/
$ sudo chmod 0700 /etc/munge/ /var/log/munge/ /var/lib/munge/
$ sudo chmod 0755 /run/munge/
$ sudo chmod 0700 /etc/munge/munge.key
$ sudo chown -R munge: /etc/munge/munge.key
```

Next, we start the munge service and configure it to start at startup.
```
$ sudo systemctl enable munge
$ sudo systemctl restart munge
```

Now, we can test the munge connection to the control node, like so:
```
$ munge -n | ssh <CONTROL_NODE> unmunge 
```
Make sure to replace `<CONTROL_NODE>` with host alias of your control node. If this is successful, you should see the munge status of the control node. If you get an error, try restarting the munge service on the control node.

## Setting up Slurm
The process to install and set up Slurm is almost the same in the control node and the worker nodes. The only significant difference is which service we have to start and enable. 
### Control Node
To install Slurm on your control node do the following. First, install the required packages with:
```
$ sudo apt install slurm-wlm
```
Then, use slurm's handy configuration file generator located at `/usr/share/doc/slurmctld/slurm-wlm-configurator.html` to create your configuration file. You can open the configurator file with your browser. 
> Slurm configuration files are a complicated topic and what values you have to fill in is specific to your machines. If you want to learn more about it, go [here](https://slurm.schedmd.com/slurm.conf.html).

You don't have to fill out all of the fields in the configuration tool since a lot of them can be left to their defaults. The following fields are the once we had to manually configure:
- ClusterName: `<YOUR-CLUSTER-NAME>`
- SlurmctldHost: `<CONTROL-NODE-NAME>`
- NodeName: `<WORKER-NODE-NAME>`[1-3] (this would mean that you have 4 worker nodes called `<WORKER-NODE-NAME>1`, `<WORKER-NODE-NAME>2`, `<WORKER-NODE-NAME>3`)
- Enter values for CPUs, Sockets, CoresPerSocket, and ThreadsPerCore according to $ lscpu (run on node computer)
- ProctrackType: linuxproc

Once you press the `submit` button at the bottom of the configuration tool a configuration file should be created at `/etc/slurm/slurm.conf`.

Now, we have to start the slurm controller node service and configure it to start at startup, like so: 
```
$ sudo systemctl enable slurmctld
$ sudo systemctl start slurmctld
```

You can now check your slurm installation is runnning and your cluster is set up with the following commands:
```
$ sudo systemctl status slurmctld # returns status of slurm service
$ sinfo		# returns cluster information
```

Once you have your worker nodes set up, you can also check the cluster is correctly set up by running:
```
$ srun -N<NUMBER-OF-NODES> hostname
```
Where `<NUMBER-OF-NODES>` is the number of worker nodes that are currently set up. If you followed all of the steps correctly, this should return the name of all of your nodes.

### Worker Nodes
We follow a similar procedure to the control node for each worker node. First, we install slurm like so:
```
$ sudo apt install slurm-wlm
```
Next, we create the configuration file. You have to make sure all the configuration files in your cluster match, so it is simpler here to just copy the `slurm.conf` configuration file from your control node. If you don't want to copy it over, then just create it the same way you did in the control node.

Now, we start the slurm worker node service and configure it to start at startup.
```
$ sudo systemctl enable slurmd
$ sudo systemctl start slurmd
```

Then, we can verify slurm is set up correctly and running like so:
```
$ sudo systemctl status slurmd
```

As long as you got no errors, your slurm worker node should now be setup. You can check that it is running correctly by using the `sinfo` or `srun` commands on your control node.

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

