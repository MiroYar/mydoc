# Настройка L2TP сервера на MicroTik роутере

## 1. Настройки пула адресов

`IP` -> `Pool`

![Pool](./img/pool.png)

## 2. Настройки профиля

`PPP` -> `Profiles`

В настройке `Remote Address` выбран пул из настройки в предыдущем пункте.

![Profiles](./img/profiles.png)

## 3. Настройки авторизации

`PPP` -> `Secrets`

В настройке `Profile` выбран профиль из настроки в предыдущем пункте.

![Secrets](./img/secrets.png)

## 4. Включение l2tp сервера

`PPP` -> `Interface` -> `L2TP Server`

![L2TP Server](./img/l2tp-server.png)

## 5. Настройки NAT

`IP` -> `Firewall` -> `NAT`

![NAT](./img/nat-01.png)
![NAT](./img/nat-02.png)

[Источник](https://vk.com/video?q=Настройка%20защищённого%20VPN%20L2TP%20на%20маршрутизаторе.&z=video-34914823_456239303%2Fpl_cat_trends)
