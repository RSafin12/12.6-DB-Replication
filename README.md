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


