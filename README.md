# SlurmDocumentation
Slurm Self Learning Documentation




# Slurm through AWS VMs

# Relevant links

Main site for the official documentation

**https://slurm.schedmd.com/**

# Setting up AWS 

Create 2 or more Ubuntu machines. I used 20.04 versions as I was having problems with the 22.04 version, use what you will. 

Just note that your Slurm version might be different according to the Ubuntu version you use. As such, check the configurator.html file that comes with your Slurm installation in order to setup your slurm.conf file.

We will go through this process a bit later on the documentation as it is not relevant for now.


# Setting up the necessary permissions and AWS keys

This section is only necessary if you are not able to use SCP to send files from one machine to another. If you are not having trouble with SSH connections and everything works properly, skip this section


Get the key file, in my case I just used the default vockey key. 
Copy the vockey key to the appropriate location and give it the necessary permissions

**nano  /home/ubuntu/.ssh/id_rsa
chmod 400 /home/ubuntu/.ssh/id_rsa**

You should be able to use SCP to copy files to where you need

Sample command. Change IP and file as needed for testing. -vvv for verbosity as it will indicate where it is trying to find the key files and if you have the correct permissions or not. Correct any errors as necessary

**scp -vvv /etc/munge/munge.key ubuntu@34.233.235.143:/home/ubuntu**

# Generating and copying munge keys



**apt update
apt install munge libmunge2 libmunge-dev**

Do this for controller and nodes

Generate a key with /usr/sbin/create-munge-key on your controller or one of your nodes. It might have already been created. 

Afterwards copy it from one machine to all others. The key should be the same across the entire cluster and controller.

The key should be sent to /etc/munge

Since you might need additional permissions to send it directly to the correct directory, it might be best to just sent it to /home/ubuntu and then copy it from there to the /etc/munge directory

Sample command:

**scp -vvv /etc/munge/munge.key ubuntu@34.233.235.143:/home/ubuntu**



# Setting up permissions and starting munge

This controls levels of access to relevant munge directories/files

**chown -R munge: /etc/munge/ /var/log/munge/ /var/lib/munge/ /run/munge/**

**chmod 0700 /etc/munge/ /var/log/munge/ /var/lib/munge/**

**chmod 0755 /run/munge/**

Enables and starts munge

**systemctl enable munge**

**systemctl start munge**


Everything should be working properly from this point. 


Before you go to the next section you might want to change the /etc/hosts file just in case

Add something like this to the bottom of the file. Change hostnames and IPs according to what you have, specifically the IP and hostname for your controller and each of your cluster nodes.

```
#Fixing addressing issues
#Change as needed
172.31.134.108 linuxcontroller linuxcontroller
172.31.142.150 linux1 linux1
```

# Slurm setup

Install slurm

**apt install slurm-wlm**

Go to **/usr/share/doc/slurmctld/slurm-wlm-configurator.html**

Copy the contents to a html file in your own local machine and then execute it in order to open a configurator in your browser. It should show something like this:





![Screenshot from 2023-07-14 10-13-55](https://github.com/AF-Github1/SlurmDocumentation/assets/133685290/ef16ad76-a791-4110-a5eb-01374ea2b9ea)



This configuration tool is very useful as it enables you to quickly generate a slurm.conf file appropriate for your Slurm version.

After getting the configuration copy the contents to **/etc/slurm-llnl/slurm.conf**

Do this for your controller and each of your nodes


You might need to change a few lines in the bottom of your configuration. Use the slurmd -C command to check what you need 

# Use slurmd -C on your nodes to check the maximum resources a node can use for jobs

Should look somewhat similar to this, values will depend on the machine

```

PartitionName=debug Nodes=linux[1] Default=YES MaxTime=INFINITE State=UP
NodeName=linux1 CPUs=1 Boards=1 SocketsPerBoard=1 CoresPerSocket=1 ThreadsPerCore=1 RealMemory=966

```
From this point everything should be functioning properly and you should be able to run jobs for your cluster


















