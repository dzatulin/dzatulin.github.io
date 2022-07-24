---
title: "GDB backtrace when PHP crashed"
date: 2022-07-23
categories:
  - Blog
tags:
  - PHP
  - System
---
Нету универсального метода узнать, что PHP дает сбой, но могут быть признаки. Периодически на PHP проектах сталкиваюсь с ситуацией, когда веб-сайт резко умирает или код работает очень медленно и внезапно получаете сообщение в браузере '"Document contains no data"' а в логе PHP-FPM ошибку такого типа '"php-fpm[26790]: segfault at 7f9d55b08638 ip 0000556d22a04348 sp 00007ffcaa6e84c0 error 4 in php-fpm[556d22826000+41a000]"' это может означать, что PHP аварийно завершает работу где-то во время выполнения кода.

Ошибка сегментации возникает из-за нарушения доступа к памяти. Значит, ошибка возникает, когда программа пытается получить доступ к блоку памяти, к которому у вас нет доступа. Или, если быть кратким, вы приближаетесь к участку памяти, который вам не принадлежит.

{% capture notice-2 %}
Короче говоря, это так же известно как **segfault**. Существуют различные причины этой ошибки:

- Не правильно сконфигурирован php-fpm. 
- Вы пытаетесь записать в ячейку памяти только для чтения.
- Вы пытаетесь получить доступ к освобожденной ячейке памяти.
- Заканчивается свободная память.
- Ошибка в коде.

<div class="notice">{{ notice-2 | markdownify }}</div>

Если вы получаете подобные ошибки в логе, вам может потребоваться создать некоторые дампы ядра, чтобы понять, что происходит.

**Core Dump** — это файл, содержащий адресное пространство (память) процесса, когда процесс неожиданно завершается.

Включение Core Dumps для PHP-FPM на CentOS 7

Set the rlimit_core directive in /etc/php-fpm.d/pool.conf and /etc/php-fpm.conf to unlimited:

rlimit_core = unlimited

Set system parameters to generate core dumps:

```
# Send dumps via a pipe to the systemd-coredump
sysctl kernel.core_pattern='| /usr/lib/systemd/systemd-coredump %p %u %g %s %t %c %e'

# Enable core dumps for processes using setuid().
sysctl fs.suid_dumpable=2 

# Do not add PID to core dump file.
sysctl kernel.core_uses_pid=0
```

Add to /etc/systemd/system.conf file:

```
DefaultLimitCORE=infinity
```

Reload systemd configuration:

```
systemctl daemon-reload
```

Restart PHP-FPM:

```
systemctl restart php-fpm
```

To interact with core dump we can use coredumpctl utility:

```
coredumpctl list

coredumpctl info 123

coredumpctl gdb 123
```