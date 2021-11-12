# Instalação do Bacula 11 utilizando Debian 10

Tutorial básico para mostrar como realizar a intalação do Bacula 11 utilizando o Debian 10.

## ATUALIZAÇÃO DO DEBIAN:

```bash
apt-get update -y && apt-get upgrade -y
```
-----------------------------

# ALTERAR NOME DE HOSTNAME E CONFIGURAR HOST:

```bash
nano /etc/hosts
```
Vamos realizar as seguintes alterações

* Comentar a linha do **localhost**

```txt
#127.0.0.1 localhost
```

* E vamos adicionar a seguinte linha (No caso você irá colocar o ip do seu servidor):

```txt
192.168.0.11	bacula
```
---

* Vamos editar o arquivo do hostname e mudar o nome da máquina.

```bash
nano /etc/hostname

bacula
```

-------------------------------------

# INSTALANDO PROGRAMAS ESSENCIAIS:

```bash
apt install wget ntp -y
```

--------------------------------------

# INSTALANDO PACOTES DE DEPENDENCIAS:

```bash
apt-get install make gcc build-essential perl unp mc mtx libreadline7 \
libreadline5-dbg libreadline-gplv2-dev zlib1g-dev lzop liblzo2-dev python-lzo sudo \
gawk gdb libacl1 libacl1-dev libssl-dev lsscsi apt-transport-https -y
```
-------------------------------------------

# INSTALAÇÃO POSTGRESQL

```bash
apt-get install postgresql-11 postgresql-contrib-11 postgresql-client-11 \
postgresql-server-dev-11 -y
```

---

# CRIAR SENHA PARA O USUARIO POSTGRES:

* Vamos entrar no usuário **postgres**

```bash
su postgres
```

* Agora acessando o banco de dados:

```bash
psql
```

* Digitaremos o comando abaixo:

```postgresql
\password
```

* Voce irá digitar a senha de sua escolha

* Logo em seguida só basta digitar o comando abaixo para sair do banco de dados.

```postgresql
\q
```

* Com o comando exit sairemos do usuário postgres e voltaremos para o root

```bash
exit
```

* Iremos realizar o restart do postgresql para que as alterações sejam aplicadas.

```bash
systemctl restart postgresql
```

------------------------------------------------

# CONFIGURAR ACESSO AO POSTGRESSQL PARA O BACULA:

* Editar o arquivo e alterar a linha abaixo:

```bash
nano /etc/postgresql/11/main/postgresql.conf 
```
* Vamos alterar as seguintes linhas.

```txt
de:    #listen_addresses = 'localhost'
para:  listen_addresses = '*'
```

* Editar o arquivo, alterar e criar as seguinte linhas:

```bash
nano /etc/postgresql/11/main/pg_hba.conf
```

* Vamos alterar as seguites linhas.

```txt
de:   local   all             postgres                                peer
para: local   all             postgres                                md5
      local   all             baculauser                              md5
```

```txt
de:   local   all             all                                     peer
para: local   all             all                                     md5    
```

* Iremos realizar o restart do serviço, para que as alterações sejam realizadas.

```bash
systemctl restart postgresql
```

-----------------------------------------------

# BAIXAR PACOTE BACULA 11:

* Vamos entrar no diretório a seguir para poder baixar os pacotes do bacula 11.

```bash
cd /usr/src
```

* Com o wget iremos baixar o Bacula 11 zipado.

```bash
wget --no-check-certificate https://sourceforge.net/projects/bacula/files/bacula/11.0.2/bacula-11.0.2.tar.gz
```

* Para descompatar basta utilizar o comando abaixo.

```bash
tar xvzf bacula-11.0.2.tar.gz
```
* Após descompactado iremos entrar no diretório do bacula para começar a fazer as configurações.

```bash
cd bacula-11.0.2
```

-------------------------------------------------

# CONFIGURAR BACULA:

* Dentro do diretório do bacula-11.0.2 iremos digitar os seguintes comandos.
    * Alterar os seguintes campos:

        * **SENHA QUE VOCE DEFINIU** - Pela senha digitada no inicio do tutorial
        * **IP DO SERVIDOR** - Para o IP que seu servidor bacula está usando.
        * **SEU EMAIL** - Para um email de sua preferencia.

```bash
./configure \
 --enable-smartalloc \
 --with-postgresql \
 --with-db-user=baculauser \
 --with-db-password=SENHA QUE VOCE DEFINIU \
 --with-db-port=5432 \
 --with-openssl \
 --with-readline=/usr/include/readline \
 --sysconfdir=/etc/bacula \
 --bindir=/usr/bin \
 --sbindir=/usr/sbin \
 --with-scriptdir=/etc/bacula/scripts \
 --with-plugindir=/etc/bacula/plugins \
 --with-pid-dir=/var/run \
 --with-subsys-dir=/etc/bacula/working \
 --with-working-dir=/etc/bacula/working \
 --with-bsrdir=/etc/bacula/bootstrap \
 --with-s3=/usr/local \
 --with-basename=bacula \
 --with-hostname=IP DO SERVIDOR \
 --with-systemd \
 --disable-conio \
 --disable-nls \
 --with-logdir=/var/log/bacula \
 --with-dump-email=SEU EMAIL \
 --with-job-email=SEU EMAIL
```

* Apos finalizado a configuração, iremos executar o comando a seguir.

```bash
make -j 8 && make install && make install-autostart
```

------------------------------------------------

# CRIAR AS TABELAS NO BACULA:

* Iremos agora para o diretório raiz e digitaremos o seguinte comando.

```bash
chmod 775 /etc/bacula
```

* Logo em seguida iremos para o diretório dos **scripts**

```bash
cd /etc/bacula/scripts
```

* Iremos digitar os seguintes comandos.

```bash
chown postgres create_postgresql_database && chown postgres make_postgresql_tables && \
chown postgres grant_postgresql_privileges && chown postgres drop_postgresql_database && \
chown postgres update_postgresql_tables
```

* Iremos agora para o usuário do **postgres**

```bash
su postgres
```

* Iremos executar os seguintes scripts.

```bash
/etc/bacula/scripts/create_postgresql_database

/etc/bacula/scripts/make_postgresql_tables

/etc/bacula/scripts/grant_postgresql_privileges
```

* Logo após iremos voltar para o root, deslogando do usuário postgres.

```bash
exit 
```

-------------------------------------------

# ALTERAR A SENHA DO USUARIO baculauser NO POSTSGRESQL:

* Agora iremos realizar a mudança do usuário **baculauser** para isso iremos retornar para o usuário do postgres.

```bash
su postgres
```

* Iremos autenticar no banco usando o seguinte comando:

```bash
psql
```

* Iremos agora digitar o seguinte comando para poder alterar a senha, onde tem SENHA alterar para a senha que vc colocou nas configurações do bacula.

```postgresql
alter user baculauser with superuser password 'SENHA';
```

* Ao finalizar vamos sair do postgresql

```postgresql
\q
```

* E iremos deslogar do usuário postgre

```bash
exit
```

* Realizaremos o restart do banco, para que as alterações sejam aplicadas.

```bash
systemctl restart postgresql
```

-------------------------------------------


# INICIAR O bconsole:

* Após tudo finalizado iremos realizar o start do bacula com o seguinte comando:

```bash
bacula start
```

* Caso tenha dado tudo certo em seguida iremos iniciar o bconsole.

```bash
bconsole
```

-----------------------------------------------

* Realizaremos o update e upgrade.

```bash
apt-get update -y && apt-get upgrade -y
```

* E por fim, realizaremos o reboot do servidor para que o servidor possa ser iniciado.

```bash
reboot
```
