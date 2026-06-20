# Прячем выходной IP за WARP

*Последний раз отредактировано: 20.06.2026*

**Это необязательная процедура.**

В этом разделе мы поднимем AmneziaWG клиент на сервере и настроим конфиг для MasterDnsVPN, чтобы он выходил в интернет через Cloudflare WARP.

Схема соединения в интернет получается следующая:

```bash
Клиент с интернет-цензурой -> DNS-резолверы -> VPS с MasterDnsVPN -> CF WARP
```

**Зачем вообще поднимать WARP?**

1. Мы прячем IP сервера VPS на случай, если какой-нибудь сайт нас не захочет пустить.

2. Мы получим IPv6 адрес (если вдруг у вас его нет) и сможем получить доступ к IPv6-only сайтам (например, ntc). Если вы используете MasterDnsVPN-GG на Android в режиме VPN, то Youtube/YT Music пытаются выйти в сеть через IPv6 адреса, вместо IPv4. Если у вас нет IPv6, то и Ютюба у вас не будет.

**Однако, у этого мероприятия есть свои недостатки:**

1. Если вдруг Cloudflare WARP ляжет, то у нас не будет выхода в интернет.

2. Некоторые вредные сайты могут нас не пускать из-под WARP (но при этом пустили бы из под родного IP VPS, например).

3. Повышается пинг из-за лишнего звена в цепочке.

**Предупреждение для читающих: предполагается, что у вас уже есть VPS на ОС Linux и настроенный сервер MasterDnsVPN. В инструкции описаны только действия, без команд. Если вы не знаете, что вводить в терминал на каком-либо этапе, загуглите, что писать.**

**Минимальное требование по опертивной памяти на VPS - 1 Гигабайт.**



Для начала, **создайте WARP конфиг**, если у вас его еще нет.

Удобнее всего создать его в Telegram-боте: [@warp_generator_bot](https://t.me/warp_generator_bot)

Либо, можно создать на сайте: [warp-mirrors.vercel.app](https://warp-mirrors.vercel.app/)

1. **Конфигурация:** AmneziaWG

2. **Протокол:** AmneziaWG 1.5, DNS сервер на ваше усмотрение, но лучше оставить Cloudflare

3. **Конечная точка:** по умолчанию. В этом случае выходная нода будет в той же стране, где находится ваша VPS.

4. Выбираем "**Все сайты**"

5. Подтверждаем генерацию.

Проверьте, работает ли конфиг, запустив из под любого клиента, который поддерживает AmneziaWG.

Если все ок, откройте конфиг в редакторе и вставьте в конец файл следующие строки:

```bash
[Socks5]
BindAddress = 127.0.0.1:7777
```

Порт на ваш вкус, можете оставить как у меня.

Отправьте конфиг на VPS по FTP, либо создайте файл (nano WARP.conf) и скопируйте/вставьте содержимое.



На VPS выполните следующие действия:

1. Скачайте билд wireproxy-awg: [Ссылка на репозиторий](https://github.com/artem-russkikh/wireproxy-awg)

2. Извлеките бинарник в удобном для вас месте. Для простоты, бинарник и конфиг расположите в одной директории.



## Одна выходная нода

**Все дальнейшие действия производятся на VPS.**

Создайте файл `warp.service` в директории `/etc/systemd/system/` и вставьте туда следующие строки:

```bash
[Unit]
Description=wireguard awg
After=network.target network-online.target

[Service]
Type=simple
WorkingDirectory=/root
ExecStart=/root/wireproxy -c config.conf
Restart=always
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

В параметрах `WorkingDirectory` и `ExecStart` укажите путь до бинарника. После ключа `-c` укажите название конфигурационного файла.

Сохраните файл, выполните след. команды по очереди:

```bash
systemctl daemon-reload
systemctl enable warp.service
systemctl start warp.service
```

Проверьте статус сервиса:

```bash
systemctl status warp.service
```

Если все сделано правильно, то сервис должен уже крутиться в фоне.

Откройте в редакторе конфигурационный файл `server_config.toml` для MasterDnsVPN.

Найдите следующие параметры и отредактируйте так:

```bash
USE_EXTERNAL_SOCKS5 = true
FORWARD_IP = "127.0.0.1"
FORWARD_PORT = 7777
```

Сохраните файл, перезапустите сервис MasterDnsVPN

```bash
systemctl restart masterdnsvpn
```

Подключитесь к серверу через удобный для вас клиент, проверьте IP адрес, например, на [2ip.io](https://2ip.io)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/warp.png?raw=true)

## Делим трафик на RU и Global

Роутить трафик на RU/не RU мы будем через связку ядро xray + 2 инстанса wireproxy. 

**Зачем это делать?** Настроив роутинг по странам, мы сможем выходить на российские сайты из-под российского IP => сайты, которые ограничивают доступ из-за рубежа, пропустят нас.

В WARP Generator нам потребуется создать еще один конфиг с выходной нодой в России. Автор бота сделал ограничение на выбор страны, нам потребуется указать IP адрес VPS.

Для этого в чате напишите `/start`, кнопка `Личный кабинет`, `Ввести IP вручную`, вставляем IP VPS.

После этого можно создать конфиг, но на этапе **Конечная точка** указываем Россию.

Отредактируйте полученный конфиг, добавив в конец  эти строки:

```bash
[Socks5]
BindAddress = 127.0.0.1:9876
```

Порт, опять же, на ваше усмотрение.

Передайте конфиг на VPS удобным для вас способом. Пусть конфиг живет рядом с бинарником wireproxy.

**Все дальнейшие действия производятся на VPS.**

Скачайте бинарник ядра xray и распакуйте его в отдельную директорию - [Ссылка на репозиторий](https://github.com/xtls/xray-core)

Скачайте geoip.dat и geosite.dat от runetfreedom, положите их в директорию рядом с xray - [Ссылка на репозиторий](https://github.com/runetfreedom/russia-blocked-geoip)

Создайте config.json и вставьте туда мой конфиг:

```bash

{
  "log": {
    "loglevel": "warning"
  },
  "dns": {
    "hosts": {
      "dns.google": [
        "8.8.8.8"
      ],
      "proxy.example.com": [
        "127.0.0.1"
      ]
    },
    "servers": [
      "1.1.1.1",
      "8.8.8.8",
      "https://dns.google/dns-query"
    ]
  },
  "inbounds": [
    {
      "tag": "socks",
      "port": 6666,
      "listen": "0.0.0.0",
      "protocol": "mixed",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ],
        "routeOnly": false
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "allowTransparent": false
      }
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "ota": false,
            "port": 9090,
            "level": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp"
      },
      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom"
    },
    {
      "tag": "block",
      "protocol": "blackhole"
    },
    {
      "tag": "5259439696584924435-proxy",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
             "address": "127.0.0.1",
            "ota": false,
            "port": 9876,
            "level": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp"
      },
      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },
    {
      "tag": "4899310626051038434-proxy",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "ota": false,
            "port": 7777,
            "level": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp"
      },
      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api"
      },
      {
        "type": "field",
        "outboundTag": "block",
        "protocol": [
          "bittorrent"
        ]
      },
      {
        "type": "field",
        "outboundTag": "block",
        "domain": [
          "geosite:category-ads-all"
        ]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "ip": [
          "geoip:private"
        ]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "domain": [
          "geosite:private"
        ]
      },
       {
        "type": "field",
        "outboundTag": "5259439696584924435-proxy",
        "ip": [
          "geoip:ru"
        ]
      },
      {
        "type": "field",
        "port": "0-65535",
        "outboundTag": "4899310626051038434-proxy"
      }
    ]
  }
}
```

Если вы меняели порты, то отредактируйте их в конфиге (напоминалка: 9876 - порт для RU эндпоинта, 7777 - эндпоинт в стране VPS).

Сохраните файл.

Создайте два .service файла в директории `/etc/systemd/system/` - для wireproxy с RU эндпоинтом и для xray.

**warp_ru.service**

```bash
[Unit]
Description=wireguard ru awg
After=network.target network-online.target

[Service]
Type=simple
WorkingDirectory=/root
ExecStart=/root/wireproxy -c config_ru.conf
Restart=always
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

**xray.service**

```bash
[Unit]
Description=xray client
After=network.target network-online.target

[Service]
Type=simple
WorkingDirectory=/root/xray
ExecStart=/root/xray/xray -c config.json
Restart=always
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

В параметрах `WorkingDirectory` и `ExecStart` укажите путь до бинарника. После ключа `-c` укажите название конфигурационного файла.

Запустите сервисы:

```bash
systemctl daemon-reload
systemctl enable warp_ru.service
systemctl enable xray.service
systemctl start warp_ru.service
systemctl start xray.service
```

Можете проверить статусы сервисов:

```bash
systemctl status warp_ru.service
systemctl status xray.service
```

Если все сделано правильно, то сервисы уже крутятся в фоне.

Откройте в редакторе конфигурационный файл `server_config.toml` для MasterDnsVPN.

Найдите следующие параметры и отредактируйте так:

```bash
USE_EXTERNAL_SOCKS5 = true
FORWARD_IP = "127.0.0.1"
FORWARD_PORT = 6666
```

Порт 6666 - inbound порт в xray.

Сохраните файл, перезапустите сервис MasterDnsVPN

```bash
systemctl restart masterdnsvpn
```

Подключитесь к серверу, проверьте IP на [2ip.io](https://2ip.io) и на [ip.mail.ru](https://ip.mail.ru). IP адреса должны различаться.



[Вернуться на главную страницу](sinekdoxa.github.io)


