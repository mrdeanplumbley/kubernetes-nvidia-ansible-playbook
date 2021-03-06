# kubernetes-nvidia-ansible-playbook
Ansible playbook for the installation of nvidia drivers, docker, nvidia docker and kubernetes with GPU support

This guide assumes you're usuing ubuntu 18.04. The playbook may need some editing if you're using other versions of ubuntu or linux

# Instructions for use

## Step 1 - Enable ssh login between the client (your pc / laptop) and the machines you wish to install the drivers + kubernetes on

In order for ansible to work you only need to enable ssh login via root using a key. You don't need to install ansible on any machine other than the one you're working on, which can be your personal computer outside your cluster.

### 1. First, on each of the machines in your cluster install and enable openssh (to allow you to ssh into the machines).

~~~~
sudo apt-get update
sudo apt-get install openssh-server
~~~~

### 2. Second, allow the ability to ssh into the machines as root

On each machine input:
~~~~
sudo passwd
~~~~

Expected output:
~~~~
[sudo] password for linuxconfig: 
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
~~~~

### 3. Third, enable ssh via root
~~~~
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo service ssh restart
~~~~

#### 4. Fouth, add a ssh key to your local machine

Generate an ssh key for your local machine, you will use this to copy to the machines on the cluster

On your local mahine enter:

~~~~
ssh-keygen -t rsa
~~~~

Expected output:

~~~~
Your identification has been saved in /Users/me/.ssh/id_rsa.
Your public key has been saved in /Users/me/.ssh/id_rsa.pub.
The key fingerprint is:
xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx me@mypc.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        E .      |
|   .   . o       |
|  o . . S .      |
| + + o . +       |
|. + o = o +      |
| o...o * o       |
|.  oo.o .        |
+-----------------+
~~~~

#### 5. Copy the new key to each of the computers in the cluster (both for your username and for root)

~~~~
ssh-copy-id username@remote_host
~~~~

~~~~
ssh-copy-id root@remote_host
~~~~

You should now be able to ssh on to each machine as root without needing to enter a passwork

try:

~~~~
ssh root@remote_host
~~~~

Where 'remote_host' is the ip or name of one of the computers in the cluster to verify

## Step 2. Install ansible on your local machine

[The installation instructions for your OS can be found here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

[You will also likely need to install python and python pip](https://www.python.org/downloads/)

You don't need to install ansible on the cluster machines

## Step 3. Clone this repo and edit the hosts file
Ansible works by sending the commands to a number of machines defined in a hosts file. Clone this repo (if you haven't already) and edit the hosts file to correspond to the ip addresses of the machines in your cluster

To test that your workstations are reachable by ansible try:

~~~~
ansible -i hosts -u root gpu_workstations -m ping
~~~~

Expected output should be:

~~~~
myworkstation | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false, 
    "ping": "pong"
}
~~~~

## Give it to me now!
If you're the type of person who likes to live life on the cutting edge, you can try and install / activate all the components at once with

~~~~
ansible-playbook -i playlists/hosts playlists/install_all.yml
~~~~

If your the type to take life as it comes, or if you already have some of these things installed (i.e nvidia drivers) then you can run them step by step in the following order. The install_all.yml file is just all these steps copy pasta into one file, so you wont miss out on anything fun.
## Step 4. Install the nvidia drivers

~~~~
ansible-playbook -i playlists/hosts playlists/install_nvidia_drivers.yml
~~~~

## Step 5. Install docker 

~~~~
ansible-playbook -i playlists/hosts playlists/install_docker.yml
~~~~

## Step 6. Install nvidia-docker 2

~~~~
ansible-playbook -i playlists/hosts playlists/install_nvidia_docker.yml
~~~~

## Step 7 Install kubernetes on all nodes

~~~~
ansible-playbook -i playlists/hosts playlists/install_kubernetes.yml
~~~~

## Step 8. Start the kubernetes cluster on master

~~~~
ansible-playbook -i playlists/hosts playlists/start_kubernetes.yml
~~~~

## Step 9. Add the nvidia device plugin

~~~~
ansible-playbook -i playlists/hosts playlists/install_gpu_device_plugin.yml
~~~~