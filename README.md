## NGIX поддержка webp и изменение размеров изображений на лету
### 1. Устанавливаем на сервер WebP
```
sudo apt-get install webp
```

### 2. Пишем скрипт web_p.sh для конвертации изображений в заданной папке
```
for f in `find /home/admin/files -type f \( -iname \*.jpeg -o -iname \*.jpg -o -iname \*.JPG -o -iname \*.png \)`
  do cwebp -quiet -q 80 ${f} -o ${f}.webp
done
```

Запускаем:
```
/bin/bash web_p.sh
```

## Открываем файл nginx.conf
В секции http добавляем поддержку webp:
```
map $http_accept $webp_suffix {
  default   "";
  "~*webp"  ".webp";
}
```

## Открываем sites-available/dafault
Описываем 2 location в нужной секции server {}

Поддержка webp:
```
location ~* ^.+\.(jpe?g|png) {
  add_header Cache-Control "public, no-transform";
  add_header Vary "Accept-Encoding";
  try_files $uri$webp_suffix $uri =404;
  expires max;
}
```

Поддержка Image-Filter:
```
location ~ ^/img([0-9]+)(?:/(.*))?$ {
   alias /home/admin/files/$2;
   image_filter_buffer 10M;
   image_filter_jpeg_quality "100";
   image_filter_interlace on;
   image_filter resize $1 -;
   expires 1y;
   add_header Pragma public;
   add_header Cache-Control "public";
}
```

Если Image-Filter не установлен, читаем инструкции https://docs.nginx.com/nginx/admin-guide/dynamic-modules/image-filter/

Проверяем конфигурацию NGINX:
```
sudo nginx -t
```

И перезапускаем:
```
sudo service nginx reload
```

Измененное изображение размером 120px будет доступно по ссылке
```
http://your-domain.com/img120/image.jpg
```
