# sechat
Simple nc based secure chat application over SSH.

Needs a SSH account on the server machine, so proper settings to create that in a chroot jail are mentioned below.
Source: http://www.58bits.com/blog/2014/01/09/ssh-and-sftp-chroot-jail


Step 1: Create your chroot directories

I chose to have the directory in /home/sechat folder

#Creating the directories
sudo mkdir -p /home/sechat/{dev,etc,lib,lib64,usr,bin,home}
sudo mkdir -p /home/sechat/usr/bin
 
#Setting owner
sudo chown root:root /home/sechat
 
#Needed for the OpenSSH ChrootDirectory directive to work
sudo chmod go-w /home/sechat


Step 2: Choose the packages you want

I am going for bash, cp, ls, clear, nc and mkdir to our jailed users (for starters).

#First the binaries
cd /home/sechat/bin
sudo cp /bin/bash .
sudo cp /bin/ls .
sudo cp /bin/cp .
sudo cp /bin/mv .
sudo cp /bin/mkdir .
sudo cp /bin/nc .
 
#Now our chjail.sh script to bring over dependencies
sudo chjail.sh /bin/bash
sudo chjail.sh /bin/ls
sudo chjail.sh /bin/cp
sudo chjail.sh /bin/mv
sudo chjail.sh /bin/mkdir
sudo chjail.sh /bin/nc

#For clear command, if you want that
cd /home/sechat/usr/bin
sudo cp /usr/bin/clear .
sudo chjail.sh /usr/bin/clear

#Add terminal info files - so that clear, and other terminal aware commands will work.
cd /home/sechat/lib
sudo cp -r /lib/terminfo .


Step 3: Create your user and jail group

Create the jail group:
sudo groupadd sshjail

Now to add new user, either use:
sudo adduser --home /home/sechat/home/username username

or copy (and then later remove) the home directory of an existing user into the home/sechat/home directory.

If you create a new user using sudo adduser --home /home/jail/home/username username - the home directory will be created in the jail, but the user's home directory in /etc/passwd will need to be edited to return it to /home/username - since the jail root will put home at the root again once the user is logged in.

Now add the user to the jail group:
sudo addgroup username jail


Step 4: Update sshd_config

We're going to edit the sshd_config file, removing the ForceCommand internal-sftp directive - since we don't want to limit our users to SFTP. Append the following to sshd_config:

Match Group jail
    ChrootDirectory /home/sechat
    X11Forwarding no
    #AllowTcpForwarding no
    AuthorizedKeysFile /home/sechat/home/%u/.ssh/authorized_keys 

We've chrooted to /home/sechat - and both SFTP and SSH logins will default to the user's home directory below the jail.
Restart the sshd daemon, and you're ready to go sudo /etc/init.d/ssh restart or service ssh restart
