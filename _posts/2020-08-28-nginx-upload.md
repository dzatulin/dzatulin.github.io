---
title: "Nginx return 500 (Internal Server Error) when uploading large files"
date: 2020-08-28
categories:
  - Blog
tags:
  - Nginx
  - System
---
При загрузке больших файлов  (>2 GB) Nginx возвращает ошибку 500 "Internal Server Error". В моем случае была проблема с свободным местом в системе. В корневом каталоге оставалось 4GB свободного пространства. Nginx сохранял по дефолту в client_body_temp и после переполнения завершал сессию. 

После добавления отдельного диска для  **client_body_temp_path** фалый начали загружаться. 
```
    client_max_body_size 3072m;
    client_body_timeout 300s;

    client_body_in_file_only clean;
    client_body_buffer_size 16K;
    client_body_temp_path /var/www/tmp;
```

Значение clean для **client_body_in_file_only** разрешает удалять временные файлы, оставшиеся по окончании обработки запроса.
**client_body_buffer_size client_header_buffer_size** - задают размер буфера(в оперативной памяти) для чтения тела и заголовка запроса клиента соответственно. Если файл превышае размер буфера в логе будет предупреждение:
```
   a client request body is buffered to a temporary file
```
Вы можете увеличивать  **client_body_buffer_size** до размера **client_max_body_size** если у вас есть резерв опеативной памяти сепециально для случайных загрузок. Это опитимизация загрузки в ОЗУ, а не в временный файл на диске.  
