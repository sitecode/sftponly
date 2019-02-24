# sftponly	

Scripts to add/del sftp-only users to a virtual hosting sytem like system like WHM. The sftp-only users assoctiated with a certain user wherein all the sftpusers files are stored. Still usses a chrooted anvironment for the sftp-only user, and connects to the assoctiated vhost user home directory via bind mount (mounted/unmounted on each login/out). Also all files are gpg encrypted on each sftp-only user logout with a specified pgp recipient.

## Getting Started

Add a new sftp-only user.

```
/home/sftponly/bin/sftponly-useradd [vhost username] [new sftp-only username]
```

Delete an existing sftp-only user.

```
/home/sftponly/bin/sftponly-userdel [vhost username] [sftp-only whole username]
```

### Installing

As root do the following.

#### 1. Setup sshd_config

Edit sshd config file:

```
vi /etc/ssh/sshd_config
```

Comment out previous Subsystem sftp line and change it to internal-sftp:

```
#Subsystem      sftp    /usr/libexec/openssh/sftp-server
Subsystem       sftp    internal-sftp
```

And add a match group for all the sftp-only users:

```
Match Group sftponly
	X11Forwarding no
	AllowTcpForwarding no
	#ChrootDirectory %h
	ChrootDirectory /var/www/sftp/%u
	MaxSessions 1
	ForceCommand internal-sftp
```

#### 2. Setup PAM sshd

Edit PAM sshd session file:

```
vi /etc/pam.d/sshd
```

Add at the bottom:

```
session    optional     pam_exec.so quiet /home/sftponly/bin/pam.d-sshd
```

#### 3. Copy over files

Copy this sftponly directory into the /home directory.

```
cd /home
```

#### 4. Install gpg recipent public key and update bin/pam.d-sshd

## Authors

* **sitecode** - *Initial work*

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* The Internet. Thank you.

## Todo

* quotas per sftp-only user
* limit only one login at a time per sftp-only user
vi /etc/security/limits.conf
        $username           -       maxlogins       1   
* encrypt with different gpg recipient per vhost user