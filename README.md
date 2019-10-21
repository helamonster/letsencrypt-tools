# letsencrypt-tools
Tools for Let's Encrypt - Renewing certificates on one machine and importing on another, etc.

# Setup on letsencrypt server:

* Create SSH key pair 

cd /root/.ssh/
ssh-keygen -t rsa -b 4096 -C root@letsencrypt.domain.com -f letsencrypt.domain.com


# Installation on pfSense

On pfSense server:

Add to /etc/rc.conf:

# see /usr/local/etc/rc.d/scponlyc
scponlyc_enable=YES


cd /usr/local/share/examples/scponly/

# patch setup_chroot.sh to fix /dev/null bug
# see https://github.com/scponly/scponly/wiki/Faq
# .. or fix manually after running setup_chroot.sh with:

[2.4.4-RELEASE][root@pfsense.domain.com]/usr/local/share/examples/scponly: mkdir  /home/le/dev
[2.4.4-RELEASE][root@pfsense.domain.com]/usr/local/share/examples/scponly: cp -a /dev/null  /home/le/dev/

# Run setup_chroot.sh (interactive)
[2.4.4-RELEASE][root@pfsense.domain.com]/usr/local/share/examples/scponly: ./setup_chroot.sh

Next we need to set the home directory for this scponly user.
please note that the user's home directory MUST NOT be writeable
by the scponly user. this is important so that the scponly user
cannot subvert the .ssh configuration parameters.

for this reason, a writeable subdirectory will be created that
the scponly user can write into.

-en Username to install [scponly]
le
-en home directory you wish to set for this user [/home/le]

-en name of the writeable subdirectory [incoming]
domains

creating  /home/le/domains directory for uploading files

Your platform (FreeBSD) does not have a platform specific setup script.
This install script will attempt a best guess.
If you perform customizations, please consider sending me your changes.
Look to the templates in build_extras/arch.
 - joe at sublimation dot org

please set the password for le:
Changing local password for le
New Password:
Retype New Password:
if you experience a warning with winscp regarding groups, please install
the provided hacked out fake groups program into your chroot, like so:
cp groups /home/le/bin/groups



# Now, create the .ssh user in the le user's home directory:

mkdir /home/le/.ssh


# Now, from le server, copy in ssh key (must ssh in as root for this):

root@letsencrypt:/etc/letsencrypt/archive/pfsense.domain.com#  scp    /root/.ssh/letsencrypt.domain.com.pub   root@pfsense.domain.com:/home/le/.ssh/authorized_keys
Password for root@pfsense.domain.com:
letsencrypt.domain.com.pub        100%  756     1.4MB/s   00:00

Test the ability to scp in as the le user with thessh key:

# test copying in keys with le user:

root@letsencrypt:/etc/letsencrypt/archive/pfsense.domain.com#  scp -i \
  /root/.ssh/letsencrypt.domain.com  {cert,chain,privkey}5.pem  le@pfsense.domain.com:domains/

cert5.pem                                                           100% 1931     3.1MB/s   00:00    
chain5.pem                                                          100% 1647     2.9MB/s   00:00    
privkey5.pem                                                        100% 1704     3.0MB/s   00:00 


# All good!
