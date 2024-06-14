# Instalando o IBM Reliable Scalable Cluster Technology (RSCT) no Ubuntu 24.04 (Noble Numbat)

Este manual explica como instalar o pacote IBM RSCT em uma LPAR Ubuntu 24 (Noble Numbat).

1. Adicione e atualize o repositório da IBM usando o gerenciador de pacotes APT:
   
   root@ubuntu:$ add-apt-repository ppa:ibmpackages/rsct
   
   root@ubuntu:$ apt update

2. Instale os pacotes do RSCT:

   root@ubuntu:$ apt install rsct.basic rsct.core rsct.core.utils rsct.opt.storagerm devices.chrp.base.servicerm libservicelog-1.1-1

3. Confirme se o pacote DynamicRM está instalado com o apt list, caso não esteja, baixe-o diretamente do repositório abaixo e instale-o com o DPKG:
 
   root@ubuntu:$ wget https://launchpad.net/~mcao-zz/+archive/ubuntu/ppa/+files/dynamicrm_2.0.7.3-21-1ubuntu1_ppc64el.deb
   
   root@ubuntu:$ dpkg -i dynamicrm_2.0.7.3-21-1ubuntu1_ppc64el.deb

4. Confira se o arquivo /opt/rsct/cfg/rmdefs/IBM.DRM.mdef existe, caso não exista, crie-o:

   root@ubuntu:$ vi /opt/rsct/cfg/rmdefs/IBM.DRM.mdef

```
   ResourceManager
 ClassKey=1
 StartCommand="/opt/rsct/bin/RMstart -s IBM.DRM -pIBM.DRMd"
 StopCommand="/opt/rsct/bin/RMstop -s IBM.DRM"
 StartTime=60
 UserName="root"
 Properties=MkDirs,AutoStart

```

5. Cerfique-se que as permissão do arquivo é 755 e o dono e grupo são do usuário root:

   root@ubuntu:$ ls -l
```
   -rwxr-xr-x 1 root root 803 Jun 23  2016 /opt/rsct/cfg/rmdefs/IBM.DRM.mdef
```
5. Reinicie o RSCT com os comandos:

   root@ubuntu:$ rmcctrl -z
   
   root@ubuntu:$ rmcctrl -A
   
   root@ubuntu:$ rmcctrl -p

## Outras dicas e comandos úteis:

### Para verificar se a conexão com o RMC está estabelecida:

   root@ubuntu:$ /opt/rsct/bin/rmcdomainstatus -s ctrmc
  ```
   Management Domain Status: Management Control Points
  
     I A  0x51e90db3370c74cf  0001  172.20.10.1
   ```
   I = Estabelecido

### O RSTC precisa contactar o HMC através da porta 657, teste se ela está aberta, por exemplo:

  root@ubuntu:~$ nc -zv 172.20.10.1 657
  ```
  Ncat: Connected to 172.20.10.1:657.
  
  Ncat: 0 bytes sent, 0 bytes received in 0.10 seconds.
  ```

  Neste exemplo usando o Netcat, 172.20.10.1 é o IP do HMC e a conexão foi estabelecida com sucesso.

### Confira se há algum firewall habilidade na LPAR como o FirewallD, UFW ou IPTables com os comandos abaixo e se estiver, crie uma regra para conexão nessa porta ou desative o firewall:

  service ufw status
  
  service firewalld status
  
  iptables -Lvn

### Se a LPAR for um clone pode ser que ela esteja com o mesmo NodeID da imagem e por isso a conexão RMC não se ativa, neste caso, altere o NodeID:

  root@ubuntu:/# /usr/sbin/rsct/bin/lsnodeid
  ```
  a4g9d52f36c7d8d1
  ```
  root@ubuntu:/# chdev -l cluster0 -a node_uuid=00000000-0000-0000-0000-000000000000
  ```
  cluster0 changed
  ```
  root@ubuntu:/# /usr/sbin/rsct/install/bin/recfgct
  ```
  /usr/lib/dr/scripts/all/ctrmc_MDdr               DR script to refresh Management Domain configuration

  0513-071 The ctcas Subsystem has been added.
  
  0513-071 The ctrmc Subsystem has been added.
  
  0513-059 The ctrmc Subsystem has been started. Subsystem PID is 9240912.
  ```
  root@ubuntu:/# /usr/sbin/rsct/bin/lsnodeid
  ```
  5d3gi415ce12f4e3
  ```
  Após alterar reinicie com o RSCT com o comando rmcctrl:

  root@ubuntu:$ rmcctrl -z
   
  root@ubuntu:$ rmcctrl -A
   
  root@ubuntu:$ rmcctrl -p
