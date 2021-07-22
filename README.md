# Configuração do servidor em CentOS 8 para Zabbix:
 
 
 
# Instalar atualizações
yum update -y
 
# Instalar utilitários
yum install -y net-tools vim nano wget curl tcpdump yum-utils sysstat ntp
 
### Instalar EPEL Release para HTOP
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install htop -y
 
# Verificar o timezone atual
timedatectl status
 
# Definir timezone
timedatectl set-timezone your_time_zone

# Para listar todos os timezone disponíveis
timedatectl list-timezones | grep your_time_zone
 
# Validar alteração do timezone
timedatectl status
 
# Verifique a data e hora atual
date
 
# Configurar regra de firewall NTP
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload
 
# Iniciar e Habilitar o NTP
systemctl enable --now ntpd
 
# Validar os ntp servers disponível
ntpq -p
 
# Sincronizar data e hora
systemctl restart ntpd
 
# Validar status SELINUX
sestatus
 
# Desabilitar SELINUX
getenforce
 
# Editar o arquivo de configuração e desabilitar o SELINUX
cat /etc/selinux/config
vim /etc/selinux/config
SELINUX=disabled
 
#É necessário reiniciar o servidor
 
# Habilitar SYSSTAT
systemctl enable --now sysstat
 
# Validar perfomance S.O
sar
 
# Ajustar Firewall MYSQL
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
 
## Instalar MySql 8 ##
#Add Repo
rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/mysql-community.repo
yum --enablerepo=mysql80-community install mysql-community-server 
 
# Habilitar serviço do MySQL
systemctl enable --now mysqld
 
# Validar serviço do MySQL
systemctl status mysqld
 
# Definir senha do usuário root do MySQL
tail -f /var/log/mysqld.log |grep password
mysql_secure_installation
 
#No banco antigo fazer backup config only
 
nohup time mysqldump -u root -pSENHA \
--flush-logs \
--single-transaction \
--create-options \
--ignore-table=zabbix.acknowledges \
--ignore-table=zabbix.alerts \
--ignore-table=zabbix.auditlog \
--ignore-table=zabbix.auditlog_details \
--ignore-table=zabbix.escalations \
--ignore-table=zabbix.events \
--ignore-table=zabbix.history \
--ignore-table=zabbix.history_log \
--ignore-table=zabbix.history_str \
--ignore-table=zabbix.history_str_sync \
--ignore-table=zabbix.history_sync \
--ignore-table=zabbix.history_text \
--ignore-table=zabbix.history_uint \
--ignore-table=zabbix.history_uint_sync \
--ignore-table=zabbix.profiles \
--ignore-table=zabbix.service_alarms \
--ignore-table=zabbix.sessions \
--ignore-table=zabbix.trends \
--ignore-table=zabbix.trends_uint \
--ignore-table=zabbix.user_history \
--ignore-table=zabbix.node_cksum zabbix >  /backup/backupconf-data.sql &
 
# No banco antigo fazer backup full
nohup time mysqldump -u root -pSENHA --single-transaction --create-options zabbix > /backup/backupfull-data.sql &
 
# Conectar no banco, criar o banco de dados e o usuário do Zabbix LOCAL
mysql -u root -p
create database zabbix character set utf8 collate utf8_bin;
create user 'zabbix'@'localhost' identified by 'SENHA';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
flush privileges;
Exit
 
#Importar banco de configuração
nohup time mysqldump -u root -pSENHA  zabbix < /backup/backupconf-data.sql &
