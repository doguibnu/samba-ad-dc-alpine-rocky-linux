# Rocky Linux 8

## Member e File Server de Arquivos do AD



""Rocky Linux é um sistema operacional comunitário desenvolvido para ser 100% compatível bug a bug com o Enterprise Linux, agora que o CentOS mudou de direção.""

Após a instalação do sistema operacional, seguir com os comandos para a instalação e configuração do samba como membro e servidor de arquivos. 

#### Atualizar o sistema:

```
yum update
```

#### Instalar os pacotes do samba necessários para samba e servidor de arquivos:

```
yum install samba samba-winbind samba-winbind-clients
```

#### Habilitar os serviços para iniciar junto ao boot do sistema:

```
systemctl enable smb nmb winbind
```

#### Configurar e editar o arquivo **/etc/hosts:**

```
nano /etc/hosts
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
10.1.1.13   SRVFILE.seu.dominio SRVFILE
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
```

Salvar o arquivo



#### Configurar e editar **/etc/hostname:**

```
nano /etc/hostname
```

```
srvfile.seu.dominio
```

Salvar o Arquivo



#### Configurar e editar **/etc/nsswitch.conf**:

```
nano /etc/nsswitch.conf
```

#### Trocar a configuração

De:

```
passwd: sss files systemd
```

**Para:**

```
passwd: files winbind
```

De:

```
group: sss files systemd
```

**Para:**

```
group: files winbind
```

Salve o arquivo 



#### Configurar a rede para inserir dados do dns do AD executando o comando:

```
nmtui
```

Escolha Editar uma** Conexão**

Digite **Enter**

Com a tela **Tab **vá em** Editar** 

Com as **teclas de direção (setas)**** mova até **Servidores DNS** e troque o **número DNS para o IP de seu AD DC** (controlador de domínio)

Ainda com as setas de direção vá até **Domínios de pesquisa** e aperte enter para adicionar o **seu.domínio**. 

Então, selecione OK **com as setas de direção e aperte enter**. 

Com a **tecla Tab **vá até a opção **voltar** e aperte **enter**. E então, com a tecla **Tab**, selecione a **opção OK **e aperte **enter**

Reiniciar o sistena:

```
reboot
```



#### Editar o arquivo **user.map** no diretório /etc/samba com o conteúdo:

```
nano /etc/samba/user.map
```

```
!root = DOMINIO\Administrator
```

Salve o arquivo



#### Configurar o arquivo **/etc/samba/smb.conf:**

```
nano /etc/samba/smb.conf
```

```
[global]  
 workgroup = DOMINIO 
 security = ADS  
 realm = SEU.DOMINIO

#logs  
 log file = /var/log/samba/%m.log  
 log level = 1

#winbind nss info  
 winbind nss info = rfc2307

#winbind  
 winbind refresh tickets = Yes  
 winbind enum users = yes  
 winbind enum groups = yes

#objects  
 vfs objects = acl_xattr  
 map acl inherit = Yes  
 store dos attributes = Yes

#keytabs  
 dedicated keytab file = /etc/krb5.keytab  
 kerberos method = secrets and keytab

#winbind dominio default  
 winbind use default domain = yes

#idmaps  
 idmap config * : backend = tdb  
 idmap config * : range = 3000-7999

#usermap  
 username map = /etc/samba/user.map

#idmaps2  
 idmap config PRUDE.PR:backend = ad  
 idmap config PRUDE.PR:schema_mode = rfc2307  
 idmap config PRUDE.PR:range = 10000-999999  
 idmap config PRUDE.PR:unix_nss_info = yes

#template para login shell e diretorio home  
 template shell = /bin/bash  
 template homedir = /home/%U

#unix primary group  
 idmap config PRUDE.PR:unix_primary_group = yes
```



#### Verificar se existe algum erro de sintaxe com o comando

```
testparm
```

Se não houver erros, executar o comando para recarregar o samba novamente

```
smbcontrol all reload-config
```

#### Reiniciar o servidor:

```
reboot
```



#### Comando para se juntar ao Domínio Criado:

```
net ads join -U administrator  
Enter administrator's password: senha do provisionamento
```

```
Using short domain name -- DOMINIO  
Joined 'FILESERVER' to dns domain 'dominio.seu'
```

## 

#### Verificar conexão NETLOGON com o AD:

```
wbinfo --ping-dc
```



#### Verificar se o serverad volta resposta com o comando: nslookup nome__do__domínio:

```
nslookup SRVAD.DOMINIO.BR
```

Por exemplo



#### E também através IP do seu Servidor AD:

```
nslookup IP-serverad
```

Se receber as respostas corretas, tudo deve estar funcionando. Caso não consiga obter as repostas, verifique as configurações novamente.

Neste momento você já deve ter uma máquina com Windows, à partir da versão 7 que possibilite se juntar em um domínio. Escolha entre windows 7 e até o 8.1 de 64 bits que **não** sejam versões **HOME EDITION**. O windows 10 infelizmente retirou a aba no RSAT onde trata dos **atributos UNIX** (**GID - Groups ID e ID - Users ID**)  e nesse caso você terá que inserir manualmente por ordem e com seu controle. Também na hora de escolher as opções que serão instaladas para o gerenciamento junto ao RSAT, não esquecer de **marcar** as opções como são mostradas [aqui]([ferramenta-nis-instalacao.png - Google Drive](https://drive.google.com/file/d/1gdxrXaqmx_bsJdzP8DxWmjAroHCud6bg/view?usp=sharing))

É necessário deixar estas opões marcadas para que você tenha acesso a aba **Unix Attributes** como [aqui]([unix-attributes1.png - Google Drive](https://drive.google.com/file/d/18BLTPKNVm_k20ph0L_hAnFNSi_wxFTVu/view?usp=sharing))

Então se o usuário **Nil** for criado através do RSAT com windows 10, você terá que inserir manualmente o valor de usuário, como:
**10001** e assim por diante. Do mesmo modo com Grupos: Grupo **TI** terá que inserir manualemente o valor: **3001**, assim como, os caminhos do diretório **home** e o **shell**.

Assumindo que você já criou o **usuário** e o **Grupo** através do **RSAT** no windows e inseriu seus atributos unix (**aba UNIX ATTRIBUTES**)  assim sendo, cada um deles possuindo seu GID (número do grupo) e ID (número do usuário), passe para póximo passo estando no servidor de arquivos (file server):

#### 

#### Verificar se retorna o GID do Grupo:

```
getent group "DOMINIO\\Domain Unix"  
domain unix:x:3006:
```



#### Garantir privilégio de acesso do Grupo Criado junto com o usuário criado no RSAT:

#### Comando:

```
net rpc rights grant "DOMINIO\Domain Unix" SeDiskOperatorPrivilege -U "DOMINIO\usuário_criado"  
Enter DOMINIO\Administrator password: senha do usuário_criado
# Caso tenha criado corretamente o sistema irá dar sucesso do comando

Successfully granted rights.
```

Chegando nesse ponto, está apto a iniciar a configuração da montagem de disco, o diretório escolhido para abrigar seus arquivos bem como, ajustar as permissões de acesso para que seja possível fazer as alterações via RSAT com as devidas permissões.

Exemplo:

Se criou um disco e disponibilizou o diretório **/srv/ad** e então, indicou esse caminho em seu **smb.conf** ou seja, após a sessão **global** indicando o nome de seu compartilhamento do servidor de arquivos, caminho, e ajustar as permissões de acesso seguindo o exemplo:

```
nano /etc/samba/smb.conf
```

```
[Rede_Arquivos]
   path = /srv/ad/
   read only = no
   browseable = yes
```

Salve o arquivo



#### Ajustando as permissões para o Grupo no diretório criado:

```
chown root:"Domain Unix" /srv/ad
chmod 755 /srv/ad
```



#### Recarregar novamente o smbcontrol

```
smbcontrol all reload-config
```

Através de seu Windows, logue-se com a conta Administrator onde Garantiu Direitos e através do RSAT inicie suas configurações desejadas de seu AD.

Espero poder ter ajudado. Muito Obrigado




