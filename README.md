# 12.6-DB-Replication

## Задание 1

## Задание 2

### По лекции на 2 VM
Воспроизвел действия лектора на 2 VM  

SHOW REPLICA STATUS\G; 
![show_slave_status_1](https://github.com/RSafin12/12.6-DB-Replication/blob/main/show_slave_status_1.png)
![show_slave_status_2](https://github.com/RSafin12/12.6-DB-Replication/blob/main/show_slave_status_2.png)

SHOW MASTER STATUS;
Создаем таблицу(до и после)
![MASTER STATUS](https://github.com/RSafin12/12.6-DB-Replication/blob/main/on_master.png)

Смотрим на реплике
таблицы не было, появилось. 
![replication](https://github.com/RSafin12/12.6-DB-Replication/blob/main/on_slave.png)

### Расширенное пояснение  
Важные директивы в конфиг  
/etc/mysql/mysql.conf.d/mysqld.cnf   
bind-address        = 127.0.0.1 - IP адрес, с которого можно подключаться к mysql. По дефолту стоит локалхост, на время практики разрешил все подключения указав 0.0.0.0, но на проде наверно так делать не стоит.  
server-id         = 1  - идентификатор с помощью которого MySQL различает сервер внутри репликации.  
log_bin                   = /var/log/mysql/mysql-bin.log - определяет имя/местоположение бинарного журнала  
binlog_do_db      = db_1  - тут можно перечислить БД для репликации, можно несколько  
binlog_do_db      = db_2  
...  
binlog_do_db      = db_3  - тут можно указать БД, которая не будет реплицироваться.  
 
Настройка пользвателя репликации на исходной БД   
Создаем пользователя   
CREATE USER 'replica'@'raven' IDENTIFIED WITH mysql_native_password BY 'pass0';  
Даруем права  
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'raven';  
FLUSH PRIVILEGES;  

Далее нужно синхр-ть мастера с репликой  
FLUSH TABLES WITH READ LOCK; - на время синка блокируем таблицы, чтобы на время синка никто не мешал.   
SHOW MASTER STATUS; - получаем данные о File и Position  

И далее 2 варианта  
а) если данных пока нет, то можно разблокировать таблицы и создать их  
UNLOCK TABLES;  
CREATE DATABASE db_1;  
exit  
б) если данные уже есть  
в отдельном окне вкладке делаем дамп с мастера  
sudo mysqldump -u root db_1 > db_1.sql   
и отправляем дамп на сервер-реплику  
scp db_1.sql username@raven:/tmp/  
После логинимся на сервере реплике, создаем там таблицу и "засасываем" дамп   
sudo mysql db < /tmp/db_1.sql  

... и правно переходим к настройке реплики.  
Настраиваем конфиг  
/etc/mysql/mysql.conf.d/mysqld.cnf   
server-id         = 2 - указываем другой ID  
relay-log           = /var/log/mysql/mysql-relay-bin.log - указываем на файл журнала ретрансляции реплики.   
ребутаем mysql.service  
systemctl restart mysql  

Запуск репликации  
Указываем, кто главный.  
и тут 2 варианта   
тоже работает(не всегда)  
CHANGE REPLICATION SOURCE TO SOURCE_HOST='strong', SOURCE_USER='replica', SOURCE_PASSWORD='pass0', SOURCE_LOG_FILE='mysql-bin.000003', SOURCE_LOG_POS=1549;   

по лекции  
CHANGE MASTER TO MASTER_HOST='strong', MASTER_USER='replica', MASTER_PASSWORD='pass0',  MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=1549;   

Разница только в префиксе, где-то в мануалах используют SOURCE, а где-то MASTER. Вероятно зависит от версии MySQL   
Активируем реплику  
START REPLICA;  
Чекаем статус  
SHOW REPLICA STATUS\G;   


Далее можно проверить работу репликации, добавив в БД таблицу на стороне мастера.   
USE db_1;  
CREATE TABLE test_table ( 1st_column varchar(30) );   
INSERT INTO test_table VALUES ('If you see that'),(' it works');  
После она должна появиться на реплике  
USE db_1;  
SHOW TABLES;  
SELECT * FROM test_table;   

Если реплика сходу не завелась, ошибка подключения и т.п., то можно попробовать ребутнуть везде mysql  
systemctl restart mysql  
и заново подсмотреть на мастере File и Pos, после заново запустить репликацию.   

![exp_1](https://github.com/RSafin12/12.6-DB-Replication/blob/main/exp_1.png)  
![exp_2](https://github.com/RSafin12/12.6-DB-Replication/blob/main/exp_2.png)  




















