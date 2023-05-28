## Prepar seu servidor

### Faça os updates do sistema

```bash
sudo dnf update
```
### Installe o editor de texto Vim
```bash
sudo dnf install vim
```
### Use o Vim para editar o arquivo de configuração fo SELinux, com isso desabilitaremos temporariamente para seguir com a instalação

```bash
sudo vim /etc/sysconfig/selinux
```
#### deentro deste arquivo encontre a linha contendo ```SELINUX=enforcing``` e troque para ```SELINUX=disabled```, com isso feito reinicie o sistema com o comando abixo

```bash
sudo reboot now
```

## Instalar Apache
### No Oracle Linux 9 assim como no Red Hat o serviço Apache é nomeado ```httpd``` para instalá-lo use o seguite comando
```bash
sudo dnf install httpd httpd-tools
```
### com o comando seguite habilitamos e iniciamos o serviço ```httpd```
```bash
sudo systemctl enable --now httpd
```

## Instalar o PHP
### Instale o PHP e os módulos necessários com o comando abaixo
```bash
sudo dnf install -y php php-xml php-session php-sockets php-ldap php-gd php-json php-mysqlnd php-gmp php-mbstring php-posix php-snmp php-intl
```

#### Uma vez instalado, teremos de modificar algumas configuraçoes para podermos usar com o cacti
#### Aprimeira coisa será modificar o arquivo ```/etc/php.ini```, neste arquivo modificaremos a variável ```date.timezone``` para usar ```Americas/Sao_Paulo``` entre outras. Primeiro abra o o arquivo usando o Vim

```bash
sudo vi /etc/php.ini
```
#### Encontre as linhas abaixo e ajuste os valores e salve o arquivo

```bash
date.timezone = Americas/Sao_Paulo
memory_limit = 512M
max_execution_time = 60
max_input_vars = 1000
```
## Instalar o MariaDB
### Usamos o ```dnf``` novamente para instalar o MariaDB, depois de instalado habilitamos e iniciamos o serviço de database
```bash
sudo dnf install -y mariadb-server mariadb
sudo systemctl enable --now mariadb
```
### Ao instalar o MariaDB o sript ```mysql_secure_installation``` será instalado automaticamente, usamos este script para aumentar a segurança em nossa instalação com o seguite comando
```bash
sudo mysql_secure_installation
```
### Ao executar o scrip acima, teremos de passar algumas respostas:
```bash
....
Enter current password for root (enter for none): 
......
Switch to unix_socket authentication [Y/n] Y
.....
Change the root password? [Y/n] Y
New password:  <Nova-senha-usuário-root>
Re-enter new password: <Nova-senha-usuário-root>
....
Remove anonymous users? [Y/n] Y
....
Disallow root login remotely? [Y/n] Y
.....
Remove test database and access to it? [Y/n] Y
......
Reload privilege tables now? [Y/n] Y
...
Thanks for using MariaDB!
```

#### Depois de configurar a segurança do MariaDB, logamos nela usando o comando mysql e criaremos um novo database para o cacti, neste caso o nome 
```bash
sudo mysql -u root -p
```
```sql
CREATE DATABASE cacti;
GRANT ALL ON cacti.* TO 'cacti'@'localhost' IDENTIFIED BY 'aloti2021#';
FLUSH PRIVILEGES;
EXIT 
```
```bash
mariadb --version
```

```bash
sudo vim /etc/my.cnf.d/mariadb-server.cnf
```

Coloque o conteudo abaixo

```bash
[mysqld]
#Configure inside [mysqld] block
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
max_heap_table_size=128M
tmp_table_size=128M
join_buffer_size=256M
innodb_buffer_pool_size=1405M
innodb_file_format=Barracuda
innodb_doublewrite=OFF
innodb_large_prefix=1
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_buffer_pool_instances=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000
```

```bash
sudo systemctl restart mariadb
```

```bash
sudo mysql -u root -p mysql < /usr/share/mariadb/mysql_test_data_timezone.sql
sudo mysql -u root -p
```
```sql
GRANT SELECT ON mysql.time_zone_name TO cacti@localhost;
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
FLUSH PRIVILEGES;
QUIT

```

```bash
sudo dnf -y install net-snmp net-snmp-utils net-snmp-libs rrdtool
sudo systemctl enable --now snmpd
```

```bash
sudo dnf install epel-release
sudo dnf install cacti -y
```


```bash
sudo mysql -u root -p cacti < /usr/share/doc/cacti/cacti.sql
```

```bash
sudo vim /usr/share/cacti/include/config.php
```



```bash
sudo vim /etc/cron.d/cacti
```

```bash
sudo vim /etc/httpd/conf.d/cacti.conf
```

```bash
sudo systemctl enable --now httpd
```

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

```bash
curl http://10.213.101.126/cacti
```
