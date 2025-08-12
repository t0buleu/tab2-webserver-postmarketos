# Новая жизнь для старого планшета: Galaxy Tab 2 с postmarketOS как веб-сервер

---

# 📱 PostmarketOS на Samsung Galaxy Tab 2 (GT-P3110) — превращение в веб-сервер

## 📖 О проекте
Этот проект посвящён модернизации устаревшего планшета **Samsung Galaxy Tab 2 (GT-P3110)** с помощью установки **postmarketOS** и его превращению в лёгкий и энергоэффективный веб-сервер.

В ходе работы было выполнено:
- 🔧 Настройка **TWRP Recovery** для прошивки через ADB sideload.
- 📦 Установка образа postmarketOS (XFCE4 + OpenRC)(есть готовые образы, но так же можно сделать свой кастомный образ через **pmbootstrap**).
- 🔐 Конфигурация удалённого доступа по **SSH**, изменение порта, настройка правил **nftables**.
- 🐳 Запуск и тестирование **Docker** и **Docker Compose** на ARMv7-устройстве.
- 🌐 Подготовка устройства к роли **мини-веб-сервера** для домашних или учебных проектов.

💡 **Идея:** показать, что старые Android-устройства можно повторно использовать, превращая их в полезные серверы с минимальным энергопотреблением.

---

# 📲 Установка образа 

- Как сделать свой кастомный образ указано здесь [pmbootstrap](https://wiki.postmarketos.org/wiki/Pmbootstrap/Using_pmbootstrap), я выбрал xfce4 так как она наиболее легкая и уже мне привычная
- Но я нашёл готовый образ на сайте [PostmarketOS для Samsung Galaxy Tab 2 7.0](https://images.postmarketos.org/bpo/v25.06/samsung-espresso7/)
- Установил через [TWRP](https://eu.dl.twrp.me/p3110/). Описание установки через TWRP не приводится — в интернете достаточно материалов.
- И дальше перезагрузка и мы видим приветственное окно имя учетной записи user, пароль 147147

---

# 📡 Настройка nftables для доступа по SSH на postmarketOS

Данный раздел описывает, как мы открыли доступ по SSH на кастомной сборке postmarketOS, установленной на Samsung Galaxy Tab 2 (GT-P3110).

## 1️⃣ Установка nftables
```bash
doas apk add nftables
```
## 2️⃣ Включение автозапуска
```bash
doas rc-update add nftables default
doas rc-service nftables start
```
## 3️⃣ Редактирование конфигурации
Открываем конфигурационный файл:
```bash
doas apk add nano
doas nano /etc/nftables.conf
```
## 4️⃣ Сама конфигурация, так же есть в репазитории:
```bash
#!/usr/sbin/nft -f

table inet filter {
    chain input {
        type filter hook input priority 0;
        policy drop;

        # Локальный трафик
        iif lo accept

        # Уже установленные соединения
        ct state established,related accept

        # SSH (порт 22 или свой)
        tcp dport 22 accept comment "Accept SSH"

        # Разрешить ping
        icmp type echo-request accept comment "Allow Ping"
    }
}
```
## 5️⃣ Применение правил
```bash
doas nft -f /etc/nftables.conf
```
## 6️⃣ Сохранение правил
```bash
doas rc-service nftables save
```








