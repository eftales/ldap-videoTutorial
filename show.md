# 目录
1. 2 min 搭建 openldap 环境
2. ldap 基础知识
3. 数据的增删减改
4. openldap 用户登录
5. phpLdapAdmin 的安装与使用
6. JNDI
7. 在 centos 上搭建 openldap 环境

- links

    https://github.com/eftales/ldap-videoTutorial


# 第一节 2 min 搭建 openldap 环境
## 你需要
1. ubuntu 16.04
2. docker
3. linux


## docker 安装脚本
```bash
# 安装 docker 和 docker-compose
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install  -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y

sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sleep 1
sudo chmod +x /usr/local/bin/docker-compose

sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker #添加docker用户组
sudo usermod -a -G docker $USER

sudo mkdir -p /etc/docker
echo "{" > daemon.json
echo "\"registry-mirrors\": [\"https://ofyci9nf.mirror.aliyuncs.com\"]" >> daemon.json
echo "}" >> daemon.json
echo "" >> daemon.json

sudo mv daemon.json  /etc/docker/daemon.json

sudo systemctl daemon-reload
sudo systemctl restart docker

sleep 5

sudo gpasswd -a $USER docker #将当前用户添加至docker用户组
sudo newgrp docker #更新docker用户组
```

## 下载 ldap 镜像
```bash
docker pull osixia/openldap:1.2.2
docker pull osixia/phpldapadmin:0.7.2
```

## 把 ldap 镜像跑起来吧~
```bash
docker run --name my-openldap-container --detach osixia/openldap:1.2.2
docker exec -it my-openldap-container bash
slaptest
ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin
```

# 第二节 openldap 基础知识
![](https://upload-images.jianshu.io/upload_images/9767009-ea2993bcdd47c1b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/9767009-a84c5c00cee097c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/9767009-a23f5c09883dbc7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/9767009-0f18e0de72a56fa3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/9767009-7b7f0dc9df18f2f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 第三节 数据的增删减改
```text
ldapmodify [ldap 服务器地址] [你的用户名] [你的密码] [ldif 文件的地址]
```

![](https://upload-images.jianshu.io/upload_images/9767009-18e6e3f7c616e057.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


file name:barbara.ldif

```ldif
dn: cn=barbara,dc=example,dc=org
objectClass: inetOrgPerson
cn: barbara
sn: Jensen
title: the world's most famous mythical manager
mail: bjensen@example.com
uid: bjensen
```

```bash
ldapadd -x -H ldap://127.0.0.2:389 -D "cn=admin,dc=example,dc=org" -w admin -f barbara.ldif
```

```bash
ldapsearch -x -H ldap://127.0.0.2:389  -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin 
```

```bash
ldapsearch -x -H ldap://127.0.0.2:389  -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin "cn=*"
```

```bash
ldapdelete -x -H ldap://127.0.0.2:389  -D "cn=admin,dc=example,dc=org" -w admin  "cn=barbara,dc=example,dc=org"
```

```ldif
dn: cn=barbara,dc=example,dc=org
changetype: add
objectClass: inetOrgPerson
cn: barbara
sn: Jensen
title: the world's most famous mythical manager
mail: bjensen@example.com
uid: bjensen
```

```bash
ldapmodify -x -H ldap://127.0.0.2:389 -D "cn=admin,dc=example,dc=org" -w admin -f barbara.ldif
```

file name: addorg.ldif
```ldif
dn: ou=People,dc=example,dc=org
changetype: add
objectclass: top
objectclass: organizationalUnit
ou: People

dn: ou=Servers,dc=example,dc=org
changetype: add
objectclass: top
objectclass: organizationalUnit
ou: Servers
```

file name: modify1.ldif
```ldif
dn: cn=barbara,dc=example,dc=org
changetype: modify
replace: title
title: one of the world's most famous mythical manager
```

file name: modify2.ldif
```ldif
dn: cn=barbara,dc=example,dc=org
changetype: modify
add: description
description: barbara description
```

file name: modify3.ldif
```ldif
dn: cn=barbara,dc=example,dc=org
changetype: modrdn
newrdn: cn=babara
deleteoldrdn: 0
newsuperior: ou=People,dc=example,dc=org
```

# 第四节 ldap 用户
ldappasswd 用于重置其他用户的密码

ldapmodify 用于更改自己的密码

```bash
ldappasswd -x -H ldap://127.0.0.2:389 -D "cn=admin,dc=example,dc=org" -w admin  "cn=barbara,dc=example,dc=org"
```

```bash
 ldappasswd -x -H ldap://127.0.0.2:389 -D "cn=barbara,dc=example,dc=org" -w xxxx -s mima
```

file name: passwd.ldif
```ldif
dn: cn=barbara,dc=example,dc=org
changetype: modify
replace: userPassword
userPassword: xinmima
```

```bash
ldapmodify -x -H ldap://127.0.0.2:389 -D "cn=barbara,dc=example,dc=org" -w mima -f passwd.ldif 
```


# 第五节 phpLdapAdmin 的安装与使用

```docker 
osixia/openldap:1.2.2
osixia/phpldapadmin:0.7.2
```


```docker-compose
version: '2'

services:
  openldap:
    container_name: openldap
    image: osixia/openldap:1.2.2
    ports:
      - "389:389"
      - "636:636"
    command: [--copy-service,  --loglevel, debug]
  phpldapadmin:
    container_name: phpldapadmin
    image: osixia/phpldapadmin:0.7.2
    ports:
      - "80:80"
    environment:
      - PHPLDAPADMIN_HTTPS="false"
      - PHPLDAPADMIN_LDAP_HOSTS=openldap
    links:
      - openldap
    depends_on:
      - openldap
```


# 第六节 JNDI
可以参考：https://blog.csdn.net/awj321000/article/details/78812358

这个不是 JNDI，用了第三方的库，个人感觉 JNDI 不好用。
