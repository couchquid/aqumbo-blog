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

### First order of business

Update the system.

```
# apt update && apt upgrade
```

Create a user for yourself.

```
# adduser username
```

Give your new user root privileges.

```
# usermod -aG sudo username
```

#### Generate SSH key

To generate a new key, enter this on your local system.

```
$ ssh-keygen
```

```
$ cat ~/.ssh/id_rsa.pub
```

```
$ su - username
```

```
$ mkdir ~/.ssh/
$ chmod 700 ~/.ssh/
```

```
$ nano ~/.ssh/authorized_keys
```
Insert your public key here.

```
$ chmod 600 ~/.ssh/authorized_keys
```

```
# nano /etc/ssh/sshd_config
```

/etc/ssh/sshd_config
```
PermitRootLogin no
PubkeyAuthentication yes
```

```
# systemctl restart ssh
```

```
$ sudo apt install ufw
```

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

```
$ sudo ufw allow 22
```

```
$ sudo ufw allow 80
```

```
$ sudo ufw allow 443
```

```
$ sudo ufw enable
```

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

```
$ sudo apt install docker-ce
```

```
$ sudo usermod -aG docker $USER
```

```
$ sudo systemctl enable docker
```

Logout

```
$ docker run hello-world
```
