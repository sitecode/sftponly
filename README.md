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

#### 1. Copy over files

Copy this repository sftponly directory into the /home directory.

```
cd /tmp
wget https://github.com/sitecode/sftponly/archive/master.zip
unzip master.zip
mv /tmp/sftponly-master/sftponly/ /home
rm /tmp/sftponly-master
cd /home
chmod 711 /home/sftponly/
chmod -R 750 /home/sftponly/bin/
```

#### 2. Setup sshd_config

Edit sshd config file:

```
vi /etc/ssh/sshd_config
```

Comment out previous Subsystem sftp line and change it to internal-sftp:

```
#Subsystem      sftp    /usr/libexec/openssh/sftp-server
#sftponly setup
Subsystem       sftp    internal-sftp
```

And add a match group for all the sftp-only users:

```
#sftponly setup
Match Group sftponly
	X11Forwarding no
	AllowTcpForwarding no
	ChrootDirectory %h
	MaxSessions 1
	ForceCommand internal-sftp
```

Restart sshd (depends on your system, CentOS 7 below):

```
/bin/systemctl restart sshd.service
```

#### 3. Setup PAM sshd

Edit PAM sshd session file:

```
vi /etc/pam.d/sshd
```

Add at the bottom:

```
session    optional     pam_exec.so quiet /home/sftponly/bin/pam.d-sshd
```

#### 4. Install gpg recipent public key and update bin/pam.d-sshd

List current keys:

```
gpg -k
```

**or** add new public key:

```
gpg --import [public key.asc]
```

**And update** encryption option with desired gpg recipient:

```
vi /home/sftponly/bin/pam.d-sshd
```

And change `[GPG RECIPIENT]` with the desired recipient identifier (i.e. email address or name).

#### 5. Other Settings

Set desired no login shell based on your system

```
vi /home/sftponly/bin/sftponly-useradd
```

## Troubleshooting

Sftp user gets logged out immediately after entering password. Maybe with error `Couldn't read packet: Connection reset by peer` or something else

* Did you choose the correct no login shell in `sftponly-useradd`? One that exists on your system? If you already created a user, go ahead and also change their login shell to the correct one.

## Authors

* **sitecode** - *Initial work*

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* The Internet. Thank you.

## Todo

* quotas per sftp-only user
* limit only one login at a time per sftp-only user
vi /etc/security/limits.conf
        $username           -       maxlogins       1   
* encrypt with different gpg recipient per vhost user
