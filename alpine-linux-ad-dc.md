### <u>Alpine Linux como AD-DC</u>

Alpine LInux é uma distro muito pequena. Sua imagem iso tem aproximadamente 140 MB. A escolha desta distro, foi pelo motivo de ser muito bem documentada e muito fácil "subir" um Controlador de domínio com ela (AD-DC). Não será abordado como baixar a imagem e como instalar. Na documentação do site da distro, tem as formas de como pode ser instalado. Opto em instalar o sistema em máquina virtual Virtualbox. 

Mudar o usuário para root:
```
su -
```


Update e Upgrade do Alpine Linux com os comandos:
```
apk update
apk upgrade --available && sync
```


Instalar os pacotes para AD-DC:
```
apk add samba-dc krb5 nano
```

Editar o arquivo **/etc/hosts**:
```
nano /etc/hosts
```
e adicionar a segunda linha com as informações de seu Host e IP do seu AD-DC ou seja, controlador de domínio:
```
127.0.0.1 srvad.seu.dominio srvad localhost.localdomain localhost
10.1.1.3 SRVAD.seu.dominio SRVAD
::1 localhost localhost.localdomai
```
Onde:

**10.1.1.3 SRVAD.seu.dominio SRVAD = IP e nome do seu Host de AD-DC**

salvar o arquivo


Editar o arquivo:
```
 nano /etc/samba/smb.conf
```

```
[global]  
 server role = domain controller  
 dns forwarder = seu_dns  
 workgroup = DOMINIO  
 realm = seu.domínio  
 netbios name = SRVAD  
 passdb backend = samba4  
 idmap_ldb:use rfc2307 = yes

 [netlogon]  
 path = /var/lib/samba/sysvol/prude.pr/scripts  
 read only = No

 [sysvol]  
 path = /var/lib/samba/sysvol  
 read only = No
```
salvar o arquivo

Onde:

dns forwarder = seu_dns  
workgroup = DOMINIO = domínio da sua empresa (por exemplo: EMPRESA)  
realm = seu.domínio  = seu.domínio
netbios name = SRVAD = nome do host de seu AD-DC


Editar o arquivo :
```
nano /etc/resolv.conf
```
```
search prude.pr
nameserver 10.1.1.3
nameserver seu_dns
```
salve o arquivo

OBS: o primeiro nameserver precisa **ser seu IP do seu host AD-DC**



Fazer o **Provisionamento** do **Samba** com o comando:
```
samba-tool domain provision --use-rfc2307 --interactive
```
```
Realm [SEU.DOMINIO]: SEU.DOMINIO
Domain [DOMINIO]: DOMINIO
Server Role (dc, member, standalone) [dc]: dc
DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]: SAMBA_INTERNAL
DNS forwarder IP address (write 'none' to disable forwarding) [seu_dns]: seu_dns1 seu_dns2
Administrator password: escolher uma senha
Retype password: repita a mesma senha
```



Fazer a cópia do arquivo **krb5.conf** para  o diretório **/etc** com o comando:
```
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```



Editar/alterar o init script do samba, copiando estes comandos e alterando no arquivo:
```
nano /etc/init.d/samba
```



Para apagar o conteúdo do arquivo utilizar o comando: **control+k** e copiar as linhas abaixo e então salvar:

```
#!/sbin/openrc-run

extra_started_commands="reload"

DAEMON=${SVCNAME#samba.}
SERVER_ROLE=`samba-tool testparm --parameter-name="server role"  2>/dev/null | tail -1`
if [ "$SERVER_ROLE" = "active directory domain controller" ]; then
        daemon_list="samba"
elif [ "$DAEMON" != "samba" ]; then
        daemon_list=$DAEMON
fi

depend() {
        need net
        after firewall
}


start_samba() {
        mkdir -p /var/run/samba
        start-stop-daemon --start --quiet --exec /usr/sbin/samba --
}

stop_samba() {
        start-stop-daemon --stop --quiet --pidfile /var/run/samba/samba.pid
}


start_smbd() {
        start-stop-daemon --start --quiet --exec /usr/sbin/smbd -- \
                ${smbd_options:-"-D"}
}

stop_smbd() {
        start-stop-daemon --stop --quiet --pidfile /var/run/samba/smbd.pid
}

start_nmbd() {
        start-stop-daemon --start --quiet --exec /usr/sbin/nmbd -- \
                ${nmbd_options:-"-D"}
}

stop_nmbd() {
        start-stop-daemon --stop --quiet --pidfile /var/run/samba/nmbd.pid
}

start_winbindd() {
        start-stop-daemon --start --quiet --exec /usr/sbin/winbindd -- \
                $winbindd_options
}

stop_winbindd() {
        start-stop-daemon --stop --quiet --pidfile /var/run/samba/winbindd.pid
}

start() {
        for i in $daemon_list; do
                ebegin "Starting $i"
                start_$i
                eend $?
        done
}

stop() {
        for i in $daemon_list; do
                ebegin "Stopping $i"
                stop_$i
                eend $?
        done
}

reload() {
        for i in $daemon_list; do
                ebegin "Reloading $i"
                killall -HUP $i
                eend $?
        done
}
```
salve o arquivo



Configurar o serviço do **samba-ad** para subir com o boot do sistema com o comando:
```
rc-update add samba
```



Restart do serviço do samba com o comando:
```
rc-service samba start
```



Reinicie seu Alpine Linux para agir como um Controlador de Domínio:
```
reboot
```


Chegando até este ponto, temos então o Controlador de Domínio funcional com o Alpine Linux.
