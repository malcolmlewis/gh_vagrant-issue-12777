# gh_vagrant-issue-12777

This repository has been created to provide the files needed to duplicate my issue with [Setting a static IP on opensuse/MicroOS fails, uses DHCP instead](https://github.com/hashicorp/vagrant/issues/12777)

### Create Ignition Iso Image
There is a pre configured file in disk/ignition the password is opensuse but requires your ssh key for remote access.

The cluster.conf file allows root login;

- Mode is 0640 or 416 in decimal

Contents are;
- PermitRootLogin yes
- PasswordAuthentication yes

A new ignition file to your requirements can be created via the [Fuel Ignition Website](https://opensuse.github.io/fuel-ignition/). You need to download the config.ign file and save in the disk/ignition directory.

Create the required Ignition iso image with the following command for a vagrant ignition iso image;

```
mkisofs -o $(pwd)/ignition.iso -V ignition disk
```

### Modify Vagrantfile

The following Vagrantfile parameters need to be set;

- Set your static IP - Default set to 192.168.10.180

- Set the Domain to use - default set to example.com

- Set your interface name - default set to enp0s20u4

Once you have modified the Vagrantfile and saved you can now run `vagrant up`

As the version is set it will pull the "16.0.0.20220225" image which is running wicked.

The system will come up and show the node name.

Check can log into the configured IP Addresss;

```
ssh root@192.168.10.180
....
....
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.10.180' (ED25519) to the list of known hosts.
gh-12777:~ # 
```
Exit and destroy the node.

```
gh-12777:~ # exit
logout
Connection to 192.168.10.180 closed.

vagrant destroy 

    gh-12777: Are you sure you want to destroy the 'gh-12777' VM? [y/N] y
==> gh-12777: Removing domain...
==> gh-12777: Deleting the machine folder
```

### Duplicate Issue 12777

Edit the Vagrantfile and rem out the version requirement and pull the latest image which is now using NetworkManager;

```
vagrant box update
vagrant up

....
==> gh-12777: Configuring and enabling network interfaces...
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

/sbin/ifdown 'ens6' || true
mv '/tmp/vagrant-network-ens6-1655340588-0' '/etc/sysconfig/network/ifcfg-ens6'
/sbin/ifup 'ens6'


Stdout from the command:



Stderr from the command:

bash: line 4: /sbin/ifdown: No such file or directory
bash: line 6: /sbin/ifup: No such file or directory
```

The box is up and running with a DHCP address and provisioning has failed.
