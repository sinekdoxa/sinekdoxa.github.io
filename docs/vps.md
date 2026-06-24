# Общие требования к доменам и VPS

*Последний раз отредактировано: 24.06.2026*

**Требования для VPS:**

- Местоположение: на ваше усмотрение

- Железо: минимум 1 ядро и 1 Гб ОЗУ

- SSD: минимум 10 Гб

- Статичный IP

На работоспособность НЕ влияют следующие ограничения:

- 16 Кб блокировка (учтите, что потребуется доступ по ssh, следовательно, если нет способа обойти эту блокировку, могут возникнуть проблемы)

- Отсутствие IP/CIDR в белых списках

Все тесты в рамках руководства проводятся на Linux машине (Ubuntu 24.04) с 1 ядром, 1 Гб ОЗУ, 20 Гб SSD, geoip Германия, 16 Кб блок подсети, на которой живет VPS.

**Где купить VPS?** На ваше усмотрение, бюджет и доступный способ оплаты. Рассмотрите варианты на сайтах:

[vdsina.ru](https://vdsina.ru/pricing/standard)

[play2go.cloud](https://play2go.cloud/)

[ru.fornex.com](https://ru.fornex.com/)

*Примечание автора репозитория: не покупалось лично, без гарантий работоспособности/99,9% аптайма.*

**Требования к доменам:**

- домен должен быть второго уровня (на случай, если вы планируете делать DNS записи средствами Cloudflare)

- домен должен быть коротким (чем короче домен, тем большей байт влезет в payload => выше скорость)

**Где покупать?** На ваше усмотрение. Если нет идей, рассмотрите варианты на сайтах:

[hostinger.com](https://hostinger.com)

[gen.xyz/number](https://gen.xyz/number)

*Примечание автора репозитория: не покупалось лично, я использовал бесплатный способ ниже.*

## Создание бесплатного домена на domain.digitalplat.org и создание DNS записей на Cloudflare

Если вы желаете остаться анонимным и/или не хотите светить паспортными данными/номерами кредитных карт при покупке домена, можно сделать домен на год бесплатно (и заодно проигнорировать два вышеуказанных требования, лол). 

Потребуется аккаунт на Github (принимает proton.me) для прохождения Know Your Customer (далее KYC).

Для этого регистрируемся на сайте:

[domain.digitalplat.org](https://domain.digitalplat.org)

После прохождения KYC создаем домен 3-го уровня, выбрав **dpdns.org**. 

Затем, регистрируется/авторизовываемся на сайте:

[dash.cloudflare.com](https://dash.cloudflare.com)

Во вкладке Domains добавляем свежесозданный домен, получаем неймсервера, прописываем их на digitalplat.org.

Затем, заходим в управление доменом в Cloudflare, открываем вкладку DNS > Records, создаем две записи:

```bash
Тип: A
Имя: короткое, например ns
Значение: IPv4 адрес сервера
Пример: ns.example.com -> 1.2.3.4
```

```bash
Тип: NS
Имя: сабдомен для туннеля, например v
Значение: ns.example.com
Пример: v.example.com -> ns.example.com
```

***При создании А записи на Cloudflare в столбике Proxy status перевести флажок в положение Off, Proxied сменится на DNS only. ***

В итоге должно получиться так:

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/cf1.png?raw=true)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/cf2.png?raw=true)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/digiplate.png?raw=true)

Спустя какое-то время (от 10 минут) к **A** домену можно будет попробовать постучаться, например, по ssh:

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/online.png?raw=true)



## Создание бесплатных домена и DNS записей на dnsexit.com

*Примечание автора репозитория: создание на dnsexit мной не проверялось, так что без гарантий работоспособности.*

*credits to @robot_Vecto_XIYIX_Vecto_robot*

**Альтернатива связке digitalplat и cloudflare**: [dnsexit.com](https://dnsexit.com/). 

Можно в рамках одного сервиса бесплатно создать домен и сделать DNS записи.

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/dnsexit1.jpg?raw=true)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/dnsexit2.jpg?raw=true)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/dnsexit3.jpg?raw=true)

![@](https://github.com/sinekdoxa/sinekdoxa.github.io/blob/main/docs/pics/dnsexit4.jpg?raw=true)



[Вернуться на  главную страницу](sinekdoxa.github.io)
