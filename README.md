# HW-12.6
# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»

---

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

При режиме master-slave все манипуляции с данными производятся на "мастере", а "слэйв" реплицирует внесённые изменения с "мастера".

Режим master-master по сути тот же master-slave, где "мастеру" добавляется роль "слэйва", и он будет реплицировать данные с того сервера, что был "слэйвом".  Работа в данном режиме - нежелательна, т.к. может привести к потерям данных.

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

#### На всех:
```
wget https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
```
```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```
```
rpm -Uvh mysql80-community-release-el7-6.noarch.rpm
```
```
yum -y install mysql-server mysql-client
```
```
mysqld --initialize
```
```
chown -R mysql: /var/lib/mysql
```
```
mkdir -p /var/log/mysql
```
```
chown -R mysql: /var/log/mysql
```
#### Добавить в /etc/my.cnf :
```
bind-address=0.0.0.0
server-id=(1..n)
log_bin=/var/log/mysql/mybin.log
```
#### Посмотреть пароль присвоенный при инициализации бд:
```
cat /var/log/mysqld.log
```
#### Сменить пароль:
```
mysql -p
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345';
```
```sql
FLUSH PRIVILEGES;
```
#### На мастере:
```sql
CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'Repl11Pass!';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
```sql
SHOW MASTER STATUS;
```
<img width="584" alt="2-1" src="https://github.com/user-attachments/assets/8f30366f-5cf0-4066-b6bb-6be2300ded7e" />

#### На слэйве:
```sql
CHANGE MASTER TO MASTER_HOST='192.168.*.*', MASTER_USER='replication', MASTER_PASSWORD='Repl11Pass!', MASTER_LOG_FILE = 'mybin.000001', MASTER_LOG_POS = (число из колонки position из статуса мастера);
```
```sql
START SLAVE;
```
```sql
SHOW SLAVE STATUS\G;
```
![2-2](https://github.com/user-attachments/assets/886d43ab-632e-41b7-93ff-980c49f1b9be)

#### На мастере: 
```sql
create database hw_12_6_Golikov;
```
#### На слэйве:
```sql
show databases;
```
<img width="545" alt="2-3" src="https://github.com/user-attachments/assets/3bd1e557-581d-4254-9e86-894d33c47db5" />

---

