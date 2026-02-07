# Домашнее задание к занятию "`Кеширование Redis/memcached`" - `Дёмин Максим`


### Задание 1

Что нужно сделать:

Кеширование

Приведите примеры проблем, которые может решить кеширование.

Приведите ответ в свободной форме.

### Решение:

Кеширование помогает решить такие проблемы:

Медленная работа из-за частых одинаковых запросов к базе/сервисам.

Пример: сайт постоянно запрашивает топ товаров или профиль пользователя. Вместо обращения в БД каждый раз - сохраняем результат в кеш и читаем оттуда быстрее.

Большая нагрузка на БД и рост затрат на инфраструктуру.

Пример: много пользователей одновременно обновляют страницу - БД перегружается. Кеш снимает часть нагрузки, потому что часть данных отдается из памяти.

Высокая задержка (latency) из-за медленных внешних API.

Пример: сервис тянет курс валют или погоду по API. Можно кешировать ответ на 1-5 минут, чтобы не дергать API постоянно и не ловить лимиты.

Пики трафика и необходимость стабилизировать систему.

Пример: во время акции/распродажи запросы резко растут. Кеш позволяет системе пережить пик, отдавая повторяющиеся данные из памяти.

Снижение количества вычислений.

Пример: тяжелые отчеты/агрегации/рендеры можно кешировать на некоторое время


### Задание 2

Что нужно сделать:

Memcached

Установите и запустите memcached.

Приведите скриншот systemctl status memcached, где будет видно, что memcached запущен.

### Решение:

1 Установить memcached: [sudo apt install -y memcached](img/01.png)

```
sudo apt install -y memcached
Следующие пакеты устанавливались автоматически и больше не требуются:
  libdrm-radeon1:i386  libgl1-amber-dri  libglapi-amber  libllvm19:i386  libuuid1:i386  libwayland-server0:i386  libxcb-dri2-0:i386  lp-solve  postgresql-common-dev
Для их удаления используйте «sudo apt autoremove».

К установке:
  memcached

Предлагаемые пакеты:
  libanyevent-perl  libcache-memcached-perl  libmemcached  libyaml-perl

Сводка:
  К обновлению: 0, К установке: 1 К удалению: 0, Не обновляется: 0
  Размер загрузки: 213 kB
  Места необходимо: 605 kB / 3 746 MB доступно

Пол:1 http://archive.ubuntu.com/ubuntu noble/main amd64 memcached amd64 1.6.24-1build3 [213 kB]
Получено 213 kB за 2с (103 kB/s)          
Выбор ранее не выбранного пакета memcached.
(Чтение базы данных … на данный момент установлено 470893 файла и каталога.)
Подготовка к распаковке …/memcached_1.6.24-1build3_amd64.deb …
Распаковывается memcached (1.6.24-1build3) …
Настраивается пакет memcached (1.6.24-1build3) …
Created symlink '/etc/systemd/system/multi-user.target.wants/memcached.service' → '/usr/lib/systemd/system/memcached.service'.
Обрабатываются триггеры для man-db (2.12.1-3) …
```

2 Включить автозапуск и запустить сервис: [sudo systemctl enable --now memcached](img/02.png)

```
sudo systemctl enable --now memcached
Synchronizing state of memcached.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable memcached
```

3 Проверить статус [systemctl status memcached / systemctl status memcached --no-pager](img/03.png)

```
systemctl status memcached --no-pager
● memcached.service - memcached daemon
     Loaded: loaded (/usr/lib/systemd/system/memcached.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-06 18:38:17 MSK; 2min 28s ago
 Invocation: f128310616774358ab4f20b3200de9f7
       Docs: man:memcached(1)
   Main PID: 3062653 (memcached)
      Tasks: 10 (limit: 17609)
     Memory: 2.2M (peak: 2.8M)
        CPU: 125ms
     CGroup: /system.slice/memcached.service
             └─3062653 /usr/bin/memcached -m 64 -p 11211 -u memcache -l 127.0.0.1 -l ::1 -P /var/run/memcached/memcached.pid

фев 06 18:38:17 maksimdemin-ThinkPad-L13 systemd[1]: Started memcached.service - memcached daemon.
```

### Задание 3

Что нужно сделать:

Удаление по TTL в Memcached

Запишите в memcached несколько ключей с любыми именами и значениями, для которых выставлен TTL 5.

Приведите скриншот, на котором видно, что спустя 5 секунд ключи удалились из базы.

### Решение:

1 Memcached текстовый протокол:

```set <key> <flags> <exptime> <bytes>```

[printf "set key1 0 5 5\r\nhello\r\nset key2 0 5 5\r\nworld\r\nget key1\r\nget key2\r\nquit\r\n" | nc 127.0.0.1 11211](img/04.png)

```
printf "set key1 0 5 5\r\nhello\r\nset key2 0 5 5\r\nworld\r\nget key1\r\nget key2\r\nquit\r\n" | nc 127.0.0.1 11211

STORED
STORED
VALUE key1 0 5
hello
END
VALUE key2 0 5
world
END
```

2 Подождать 6 секунд: [sleep 6](img/05.png)

3 Проверить, что ключи исчезли: [printf "get key1\r\nget key2\r\nquit\r\n" | nc 127.0.0.1 11211](img/05.png)

```
sleep 6
printf "get key1\r\nget key2\r\nquit\r\n" | nc 127.0.0.1 11211

END
END
```





### Задание 4

Что нужно сделать:

 Запись данных в Redis
 
Запишите в Redis несколько ключей с любыми именами и значениями.

Через redis-cli достаньте все записанные ключи и значения из базы, приведите скриншот этой операции.

### Решение:

1 Установить Redis [sudo apt install -y redis-server](img/06.png)
```
sudo apt install -y redis-server
Следующие пакеты устанавливались автоматически и больше не требуются:
  libdrm-radeon1:i386  libgl1-amber-dri  libglapi-amber  libllvm19:i386  libuuid1:i386  libwayland-server0:i386  libxcb-dri2-0:i386  lp-solve  postgresql-common-dev
Для их удаления используйте «sudo apt autoremove».

К установке:
  redis-server

Зависимости к установке:
  libjemalloc2  redis-tools

Предлагаемые пакеты:
  ruby-redis

Сводка:
  К обновлению: 0, К установке: 3 К удалению: 0, Не обновляется: 0
  Размер загрузки: 1 474 kB
  Места необходимо: 7 529 kB / 3 725 MB доступно

Пол:1 http://archive.ubuntu.com/ubuntu noble/universe amd64 libjemalloc2 amd64 5.3.0-2build1 [256 kB]
Пол:2 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 redis-tools amd64 5:7.0.15-1ubuntu0.24.04.2 [1 166 kB]
Пол:3 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 redis-server amd64 5:7.0.15-1ubuntu0.24.04.2 [51,7 kB]
Получено 1 474 kB за 3с (554 kB/s)         
Выбор ранее не выбранного пакета libjemalloc2:amd64.
(Чтение базы данных … на данный момент установлено 470924 файла и каталога.)
Подготовка к распаковке …/libjemalloc2_5.3.0-2build1_amd64.deb …
Распаковывается libjemalloc2:amd64 (5.3.0-2build1) …
Выбор ранее не выбранного пакета redis-tools.
Подготовка к распаковке …/redis-tools_5%3a7.0.15-1ubuntu0.24.04.2_amd64.deb …
Распаковывается redis-tools (5:7.0.15-1ubuntu0.24.04.2) …
Выбор ранее не выбранного пакета redis-server.
Подготовка к распаковке …/redis-server_5%3a7.0.15-1ubuntu0.24.04.2_amd64.deb …
Распаковывается redis-server (5:7.0.15-1ubuntu0.24.04.2) …
Настраивается пакет libjemalloc2:amd64 (5.3.0-2build1) …
Настраивается пакет redis-tools (5:7.0.15-1ubuntu0.24.04.2) …
Настраивается пакет redis-server (5:7.0.15-1ubuntu0.24.04.2) …
Created symlink '/etc/systemd/system/redis.service' → '/usr/lib/systemd/system/redis-server.service'.
Created symlink '/etc/systemd/system/multi-user.target.wants/redis-server.service' → '/usr/lib/systemd/system/redis-server.service'.
Обрабатываются триггеры для man-db (2.12.1-3) …
Обрабатываются триггеры для libc-bin (2.40-1ubuntu3.1) …
```

2 Запустить и включить автозапуск: [sudo systemctl enable --now redis-server](img/07.png)
```
sudo systemctl enable --now redis-server
Synchronizing state of redis-server.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable redis-server
```
3 Открыть redis-cli: [redis-cli](img/08.png)
```
redis-cli  
127.0.0.1:6379>
```

4 Записать несколько ключей [SET name "Maxim "SET city "SPb" SET course "Netology-Redis"](img/09.png)
```
redis-cli
127.0.0.1:6379> SET name "Maxim"
OK
127.0.0.1:6379> SET city "SPb"
OK
127.0.0.1:6379> SET course "Netology-Redis"
OK
127.0.0.1:6379>
```

5 Посмотреть все ключи: KEYS *

6 Посмотреть значения: [MGET name city course](img/10.png)
```
127.0.0.1:6379> KEYS *
1) "course"
2) "city"
3) "name"
127.0.0.1:6379> MGET name city course
1) "Maxim"
2) "SPb"
3) "Netology-Redis"
```
7 Выйти [exit](img/11.png)


### Задание 5

Что нужно сделать:

Работа с числами

Запишите в Redis ключ key5 со значением типа "int" равным числу 5. Увеличьте его на 5, чтобы в итоге в значении лежало число 10.

Приведите скриншот, где будут проделаны все операции и будет видно, что значение key5 стало равно 10.

### Решение:

1 ключ key5 со значением 5: SET key5 5

2 увеличить на 5: INCRBY key5 5

3 проверить значение: [GET key5](img/12.png)

```
redis-cli
127.0.0.1:6379> SET key5 5
OK
127.0.0.1:6379> INCRBY key5 5
(integer) 10
127.0.0.1:6379> GET key5
"10"
127.0.0.1:6379> exit
```
