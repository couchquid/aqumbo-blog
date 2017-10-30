+++
date = "2017-10-30T10:18:21+02:00"
title = "Install a Debian 9 server with Docker"
type = "post"
+++

#### Introduction

You need a server, here are some recommendations:

[Vultr](https://www.vultr.com/?ref=7127950), [Digital Ocean](https://m.do.co/c/7a06d34d7dbc), [Linode](https://www.linode.com/?r=65f9a6e1ce5187febb45bd4537e22d55d21787d0)

You will get a mail with the server's IP address and root password.

```
$ ssh root@YOUR_SERVER_IP_ADDRESS
```

Accept the warning about host authenticity.

#### First order of business

Update the system.

    # apt update && apt upgrade

Create a user for yourself.

    # adduser username

Give your new user root privileges.

```
# usermod -aG sudo username
```

#### Generate SSH key

To generate a new key, enter this on your local system.

```
$ ssh-keygen
```
Print out your key and copy it.

```
$ cat ~/.ssh/id_rsa.pub
```

Switch to your user.

```
$ su - username
```
Create the .ssh directory.

```
$ mkdir ~/.ssh/
$ chmod 700 ~/.ssh/
```
Insert your public key using nano.

```
$ nano ~/.ssh/authorized_keys
```
Change permissions.

```
$ chmod 600 ~/.ssh/authorized_keys
```
Edit the SSH daemons settings.

```
# nano /etc/ssh/sshd_config
```
Don't accept remote root login and require a key.

/etc/ssh/sshd_config
```
PermitRootLogin no
PubkeyAuthentication yes
```
Restart SSH.

```
# systemctl restart ssh
```
#### UFW Firewall

Install UFW to edit your firewall.

```
$ sudo apt install ufw
```
Deny all incoming and allow all outgoing.

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```
Allow SSH.

```
$ sudo ufw allow 22
```
Allow HTTP

```
$ sudo ufw allow 80
```
Allow HTTPS

```
$ sudo ufw allow 443
```
Turn on the firewall.

```
$ sudo ufw enable
```
#### Docker

Install the Docker repository.
```
$ sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
```

```
$ curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
```

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
```

```
$ sudo apt update
```
Install Docker

```
$ sudo apt install docker-ce
```
To be able to use docker without being root, add yourself to the docker group.

```
$ sudo usermod -aG docker $USER
```
Enable Docker at boot.

```
$ sudo systemctl enable docker
```

Logout and login again.
Then try if Docker work.

```
$ docker run hello-world
```
