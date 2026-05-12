# Домашнее задание к занятию "`Репликация и масштабирование. Часть 1`" - `Манукян Степан`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

`Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия..`

### Ответ

```
В режиме master-slave все операции на добавление, изменение и удаление проходят с master базой. Операции выборки со slave базой. История операций с master базой копируется в slave и воспроизводится для актулизации состояния. Подходит для повышенных операций на чтение.

В режиме master-master все операции производятся одинаково и синхронизируются между базами. Подходит для повышения отказоустойчисвости.
```

---

### Задание 2

`Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.
Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.`

docker-compose
```
services:
  mysql-master:
    image: mysql:8.0
    container_name: mysql-master
    environment:
      MYSQL_ROOT_PASSWORD: "1234"
      MYSQL_DATABASE: "test_db"
    volumes:
      - ./master/my.cnf:/etc/mysql/my.cnf
      - ./master/data:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - mysql-replication

  mysql-slave:
    image: mysql:8.0
    container_name: mysql-slave
    environment:
      MYSQL_ROOT_PASSWORD: "1234"
    volumes:
      - ./slave/my.cnf:/etc/mysql/my.cnf
      - ./slave/data:/var/lib/mysql
    ports:
      - "3307:3306"
    depends_on:
      - mysql-master
    networks:
      - mysql-replication

networks:
  mysql-replication:
    driver: bridge
```
master
```
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW
binlog_do_db = test_db
```
slave
```
[mysqld]
server-id = 2
log_bin = mysql-bin
relay_log = mysql-relay-bin
read_only = 1
```
Настройка на мастер ноде
```
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'repl_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

SHOW MASTER STATUS;
```
![MASTER](/img/Screenshot_4.png)

Настройка на slave ноде
```
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST = 'mysql-master',
  SOURCE_PORT = 3306,
  SOURCE_USER = 'repl_user',
  SOURCE_PASSWORD = 'repl_password',
  SOURCE_LOG_FILE = 'mysql-bin.000003',
  SOURCE_LOG_POS = 943,
  GET_SOURCE_PUBLIC_KEY = 1;

START REPLICA;
```
![SLAVE](/img/Screenshot_5.png)

Проверка
`SHOW REPLICA STATUS\G`   

![FINAL](/img/Screenshot_5.png)
---
