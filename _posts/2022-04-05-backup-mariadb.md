---
title: "Mariabackup backup and restore big data"
date: 2022-04-05
categories:
  - Blog
tags:
  - MariaDB
---

**Mariabackup** Механизм резервирования базы данных MariaDB c полной поддержкой всех функций(сжатие страниц InnoDB, шифрование данных в состоянии покоя). Пакет Mariabackup доступен с версии MariaDB 10.1. Mariabackup, который основан на хорошо известном и широко используемом инструменте резервного копирования Percona XtraBackup.
   Mariabackup поддерживает полное резервное копирование, инкрементальное и частичное:
   - Полные резервные копии создают полную резервную копию сервера базы данных в пустом каталоге, в то время как добавочные резервные копии обновляют предыдущую резервную копию с учетом любых изменений данных, произошедших с момента резервного копирования.
   - Инкрементное резервное копирование отслеживает изменения страниц из последней полной резервной копии и копирует дельта-страницы.
   - Частичное позволяет вам выбирать, какие базы данных или таблицы для резервного копирования, если задействованная таблица или раздел находятся в табличном пространстве InnoDB «файл на таблицу» (не в ibdata1).

Предположим, что мы делаем резервную копию на **server01** и разворачиваем на **server02**. Максимальный размер БД с которым мне приходилось работать это 5.2ТБ. Для надежного выполнения команд лучше запускать в фоне(screen или tmux), лично для меня tmux более user-friendly.

Выполняем полный бекап c **server01** на **server02**:
```
mariabackup --user=root --password=******** --parallel=10 --backup --compress --compress-threads=16 --stream=xbstream ssh root@server02 "mbstream -x -C /var/mysql/.backup"
```
Подготовка сервера **server02**: 
```
systmectl mariadb stop
rm -rf /var/lib/mysql/* 
rm -rf /tmp/mysql/*
rm -rf /var/log/mysql/*
```
Выполняем декомпрессию **server02**:
```
mariabackup --user=root --password=******** --parallel=16 --decompress --remove-original --target-dir=/var/mysql/.backup
```
Провека и подготовка бекапа **server02**: 
```
mariabackup --prepare --user=root --password=******** --target-dir=/var/mysql/.backup
```
Восстанавливаем бекап **server02**:
```
mariabackup --move-back --force-non-empty-directories --user=root password=********  --target-dir=/var/mysql/.backup
```
Восстанавливаем права и стартуем сервер **server02**:
```
chmod 755 /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
chown -R mysql:mysql /var/log/mysql
chown -R mysql:mysql /tmp/mysql
find /var/lib/mysql -type d -exec chmod 750 {} \;
systemctl start mariadb
```
