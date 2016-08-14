

## 1. Create the keypair on the client machine
Instructions from https://www.howtoforge.com/linux-basics-how-to-install-ssh-keys-on-the-shell
```
ssh-keygen -t rsa
```
Hit enter to accept defaults for file location and passphrase (no passphrase)

## 2. Edit the Synology SSH Daemon config file to allow pulbic key authentication
https://www.chainsawonatireswing.com/2012/01/15/ssh-into-your-synology-diskstation-with-ssh-keys
Edit the file `/etc/ssh/sshd_config` in the following lines:
edit...
```
#RSAAuthentication yes
#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
#AuthorizedKeysFile .ssh/authorized_keys
```
to...
```
#RSAAuthentication yes
PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile  .ssh/authorized_keys
```

## 3. Create the keys on server
From an open SSH connection to NAS:
```
$ cd ~
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

## 4. Copy the keys from client to server
```
$ cat ~/.ssh/id_rsa.pub | ssh user@192.168.1.10 "cat >> ~/.ssh/authorized_keys"
```

## 5. Edit client config file
Edit `~/.ssh/config`
```
# Synology DiskStation NAS
Host ds
HostName 192.168.1.10
Port 22
User username
```
