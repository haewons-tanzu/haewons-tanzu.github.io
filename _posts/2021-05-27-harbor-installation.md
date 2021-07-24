---
layout: post
title: Setting up Harbor for installing TKG on Air-Gapped Environment
categories: [TKG, Harbor, Docker]
---

## Environment
- Docker: 20.10.2
- Docmer Compose: 1.29.2
- Harbor: Spring Boot 2.3.1
- VMWare Tanzu Kubernetes Grid 1.3.1
- OS: Ubuntu 18.04.5 LTS

## Goal
A private registry is required to install TKG in an air-gapped environment. This is written to use the Harbor as a 
private registry for installing TKG in an air-gapped environment.

## Preparation
Before installing Harbor, we need to install Docker and Docker Compose.

#### 1. Installing Docker
```shell
$ sudo apt install docker.io
```
For more information, please refer this [link](https://docs.docker.com/engine/install/ubuntu/).

#### 2. Installing Docker Compose
I installed Docker Compose version 1.29.2, but you can change this version as you wish.
```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
$ sudo chmod +x /usr/local/bin/docker-compose 
```

#### 3. Grant authorization for current user
```shell
$ sudo groupadd docker 
$ sudo usermod -aG docker $USER 
$ newgrp docker 
```

## Harbor Installation

#### 1. Download Harbor Distribution
You can download harbor release [here](https://github.com/goharbor/harbor/releases).

```shell
$ tar -xvzf harbor-offline-installer-v2.2.3.tgz 
```

You can see harbor directory.
```shell
$ cd harbor
```

#### 2. Generating certificates for Harbor
We'll use Harbor hostname with IP address, not FQDN.\Please check your current VM's IP.
```shell
$ ip addr
```

Generate certificates in your harbor directory.
```shell
$ rm -rf ca.* server.* v3.ext 
$ sudo docker-compose down 

$ openssl genrsa -out ca.key 4096 
$ touch ~/.rnd 
$ openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=US/ST=CA/L=Palo Alto/O=VMware/OU=MAPBU/CN=10.0.0.4" -key ca.key -out ca.crt 

$ openssl genrsa -out server.key 4096 
$ openssl req -sha512 -new -subj "/C=US/ST=CA/L=Palo Alto/O=VMware/OU=MAPBU/CN=10.0.0.4" -key server.key -out server.csr 
```

```shell
$ cat > v3.ext <<EOF
authorityKeyIdentifier=keyid,issuer 
basicConstraints=CA:FALSE 
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment 
extendedKeyUsage = serverAuth 
subjectAltName = @alt_names 

[alt_names] 
IP.1=10.0.0.4 
EOF
```

```shell
$ openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt 
$ sudo mkdir -p /data/cert 
$ sudo cp server.crt server.key /data/cert/ 
$ openssl x509 -inform PEM -in server.crt -out server.cert 

$ sudo mkdir -p /etc/docker/certs.d/10.0.0.4 
$ sudo cp server.cert /etc/docker/certs.d/10.0.0.4/ 
$ sudo cp server.cert /etc/docker/certs.d/10.0.0.4/server.crt 
$ sudo cp server.key /etc/docker/certs.d/10.0.0.4/ 
$ sudo cp ca.crt /etc/docker/certs.d/10.0.0.4/ 

$ sudo mkdir -p /data/ca_download 
$ sudo cp ca.crt /data/ca_download 

$ sudo /usr/bin/c_rehash /data/cert/ 
$ sudo systemctl restart docker 
```

#### 3. Configuring Harbor configuration
Copy harbor.yml from template file and modify as below.
The point is to change hostname and certificate location with your environment. In my environment, I have 10.0.0.4 
as IP address.

```yaml
hostname: 10.0.0.4

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /etc/docker/certs.d/10.0.0.4/server.crt
  private_key: /etc/docker/certs.d/10.0.0.4/server.key
```

#### 4. Installing Harbor
Execute as shell script in harbor directory.
```shell
$ sudo ./install.sh
```

#### 5. Check if Harbor is running
```shell
$ docker ps
```

All done. Let's install TKG. :)
