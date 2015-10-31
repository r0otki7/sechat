# sechat
Simple nc based secure chat application over SSH.

Needs a SSH account on the server machine, so proper settings to create that in a chroot jail are mentioned below.
Source: http://www.58bits.com/blog/2014/01/09/ssh-and-sftp-chroot-jail


#Step 1: Create your chroot directories

I chose to have the directory in /home/sechat folder

##Creating the directories
_sudo mkdir -p /home/sechat/{dev,etc,lib,lib64,usr,bin,home}_
_sudo mkdir -p /home/sechat/usr/bin_
 
##Setting owner
_sudo chown root:root /home/sechat_
 
##Needed for the OpenSSH ChrootDirectory directive to work
_sudo chmod go-w /home/sechat_


#Step 2: Choose the packages you want

I am going for bash, cp, ls, clear, nc and mkdir to our jailed users (for starters).

##First the binaries
_cd /home/sechat/bin_
_sudo cp /bin/bash ._
_sudo cp /bin/ls ._
_sudo cp /bin/cp ._
_sudo cp /bin/mv ._
_sudo cp /bin/mkdir ._
_sudo cp /bin/nc ._
 
##Now our chjail.sh script to bring over dependencies
_sudo chjail.sh /bin/bash_
_sudo chjail.sh /bin/ls_
_sudo chjail.sh /bin/cp_
_sudo chjail.sh /bin/mv_
_sudo chjail.sh /bin/mkdir_
_sudo chjail.sh /bin/nc_

##For clear command, if you want that
_cd /home/sechat/usr/bin_
_sudo cp /usr/bin/clear ._
_sudo chjail.sh /usr/bin/clear_

##Add terminal info files - so that clear, and other terminal aware commands will work.
_cd /home/sechat/lib_
_sudo cp -r /lib/terminfo ._


#Step 3: Create your user and jail group

##Create the jail group:
_sudo groupadd sshjail_

Now to add new user, either use:
_sudo adduser --home /home/sechat/home/username username_

or copy (and then later remove) the home directory of an existing user into the home/sechat/home directory.

If you create a new user using _sudo adduser --home /home/sechat/home/username username_ - the home directory will be created in the sechat, but the user's home directory in /etc/passwd will need to be edited to return it to /home/username - since the jail root will put home at the root again once the user is logged in.

Now add the user to the jail group:
_sudo addgroup username sshjail_


#Step 4: Update sshd_config

We're going to edit the sshd_config file, removing the ForceCommand internal-sftp directive - since we don't want to limit our users to SFTP.
Append the following to sshd_config:

_Match Group sshjail_
    _ChrootDirectory /home/sechat_
    _X11Forwarding no_
    _#AllowTcpForwarding no_
    _AuthorizedKeysFile /home/sechat/home/%u/.ssh/authorized_keys_
    
We've chrooted to /home/sechat - and both SFTP and SSH logins will default to the user's home directory below the jail.
Restart the sshd daemon, and you're ready to go sudo /etc/init.d/ssh restart or service ssh restart
