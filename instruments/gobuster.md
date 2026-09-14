# 🚪 Gobuster — шпаргалка

---

## 📌 Оглавление
- [1. Что такое Gobuster](#-1-что-такое-gobuster)
- [2. Установка](#-2-установка)
- [3. Базовый синтаксис](#-3-базовый-синтаксис)
- [4. Режимы работы](#-4-режимы-работы)
- [5. Режим dir (директории и файлы)](#-5-режим-dir-директории-и-файлы)
- [6. Режим dns (поддомены)](#-6-режим-dns-поддомены)
- [7. Режим vhost (виртуальные хосты)](#-7-режим-vhost-виртуальные-хосты)
- [8. Режим s3 / gcs (облачные бакеты)](#-8-режим-s3--gcs-облачные-бакеты)
- [9. Режим fuzz (фаззинг)](#-9-режим-fuzz-фаззинг)
- [10. Общие флаги](#-10-общие-флаги)
- [11. Готовые команды для пентеста](#-11-готовые-команды-для-пентеста)
- [12. Словари (wordlists)](#-12-словари-wordlists)
- [13. Что считать красным флагом](#-13-что-считать-красным-флагом)
- [14. Сравнение с ffuf и feroxbuster](#-14-сравнение-с-ffuf-и-feroxbuster)
- [15. Типичные ошибки](#-15-типичные-ошибки)
- [16. Полезные ссылки](#-16-полезные-ссылки)

---

## 📌 1. Что такое Gobuster

**Gobuster** — быстрый инструмент на языке **Go** для брутфорса:
- директорий и файлов на веб-сервере (`dir`);
- DNS-поддоменов (`dns`);
- виртуальных хостов (`vhost`);
- облачных бакетов S3/GCS (`s3`, `gcs`);
- произвольных параметров (`fuzz`).

Работает многопоточно, поэтому быстрее многих аналогов. **Не парсит HTML** — просто перебирает слова из словаря и смотрит на ответы сервера.

---

## 📌 2. Установка

```bash
# Kali / Debian / Ubuntu
sudo apt update && sudo apt install gobuster -y

# Через Go (свежая версия)
go install github.com/OJ/gobuster/v3@latest

# Проверка
gobuster version
```

---

## 📌 3. Базовый синтаксис

```bash
gobuster [режим] [флаги]
```

Пример:
```bash
gobuster dir -u http://target.com -w wordlist.txt -t 50
```

---

## 📌 4. Режимы работы

| Режим | Что делает |
|-------|------------|
| `dir` | Поиск директорий и файлов |
| `dns` | Поиск DNS-поддоменов |
| `vhost` | Поиск виртуальных хостов |
| `s3` | Поиск открытых S3-бакетов (AWS) |
| `gcs` | Поиск Google Cloud Storage бакетов |
| `fuzz` | Генерический фаззинг |
| `tftp` | Поиск файлов на TFTP-серверах |

---

## 📌 5. Режим `dir` (директории и файлы)

### Основные флаги

| Флаг | Описание |
|------|----------|
| `-u <URL>` | Целевой URL (обязательно) |
| `-w <file>` | Словарь (обязательно) |
| `-x <ext>` | Расширения файлов: `php,html,txt,bak,zip` |
| `-t <N>` | Потоки (по умолчанию 10) |
| `-s <коды>` | Показывать только эти статус-коды: `200,301,403` |
| `-b <коды>` | Исключить статус-коды |
| `-r` | Следовать редиректам |
| `-k` | Игнорировать ошибки TLS |
| `-c <cookie>` | Передать cookie |
| `-H <header>` | Кастомный заголовок |
| `-a <UA>` | User-Agent |
| `-p <proxy>` | Прокси (например, Burp) |
| `-o <file>` | Сохранить результат |
| `-q` | Тихий режим |
| `--exclude-length <N>` | Исключить ответы указанной длины |
| `--timeout <sec>` | Таймаут запроса |
| `--no-error` | Не показывать ошибки |
| `--wildcard` | Принудительно обрабатывать wildcard-ответы |
| `--random-agent` | Случайный User-Agent |

### Примеры

```bash
# Базовый поиск директорий
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# С расширениями файлов
gobuster dir -u http://target.com -w common.txt -x php,html,txt,bak,zip,sql,env

# Только интересные статус-коды
gobuster dir -u http://target.com -w common.txt -s 200,301,302,403

# С куками (аутентифицированная зона)
gobuster dir -u http://target.com -w common.txt -c "session=abc123"

# Кастомный User-Agent (обход WAF)
gobuster dir -u http://target.com -w common.txt -a "Mozilla/5.0 (Windows NT 10.0)"

# Через Burp Suite
gobuster dir -u http://target.com -w common.txt -p http://127.0.0.1:8080

# Исключить soft-404
gobuster dir -u http://target.com -w common.txt --exclude-length 1234

# Сохранить результат
gobuster dir -u http://target.com -w common.txt -o result.txt
```

---

## 📌 6. Режим `dns` (поддомены)

### Основные флаги

| Флаг | Описание |
|------|----------|
| `-d <domain>` | Целевой домен |
| `-w <file>` | Словарь поддоменов |
| `-i` | Показывать IP-адреса найденных поддоменов |
| `-r <resolver>` | Кастомный DNS-сервер |
| `--wildcard` | Обработка wildcard DNS |
| `-t <N>` | Потоки |
| `-o <file>` | Сохранить результат |

### Примеры

```bash
# Поиск поддоменов
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50

# С IP-адресами
gobuster dns -d example.com -w subdomains.txt -i

# С кастомным DNS (например, 8.8.8.8)
gobuster dns -d example.com -w subdomains.txt -r 8.8.8.8

# Обработка wildcard
gobuster dns -d example.com -w subdomains.txt --wildcard
```

---

## 📌 7. Режим `vhost` (виртуальные хосты)

### Основные флаги

| Флаг | Описание |
|------|----------|
| `-u <URL>` | Целевой URL |
| `-w <file>` | Словарь имён хостов |
| `--append-domain` | Добавлять домен к каждому слову |
| `--domain <domain>` | Домен для добавления |
| `-t <N>` | Потоки |
| `-o <file>` | Сохранить результат |

### Примеры

```bash
# Базовый поиск vhost
gobuster vhost -u http://target.com -w vhosts.txt

# С автоматическим добавлением домена
gobuster vhost -u http://10.10.10.50 -w subdomains.txt --append-domain --domain target.com

# С кастомным заголовком Host
gobuster vhost -u http://10.10.10.50 -w vhosts.txt -H "Host: FUZZ.target.com"
```

---

## 📌 8. Режим `s3` / `gcs` (облачные бакеты)

### Флаги

| Флаг | Описание |
|------|----------|
| `-w <file>` | Словарь имён бакетов |
| `-t <N>` | Потоки |
| `-o <file>` | Сохранить результат |

### Примеры

```bash
# Поиск открытых S3-бакетов
gobuster s3 -w /usr/share/seclists/Discovery/Cloud/Bucket-Names.txt

# Поиск Google Cloud Storage
gobuster gcs -w bucket-names.txt
```

---

## 📌 9. Режим `fuzz` (фаззинг)

Используется для подстановки `FUZZ` в URL, заголовки, cookies.

### Примеры

```bash
# Фаззинг пути
gobuster fuzz -u http://target.com/FUZZ -w wordlist.txt

# Фаззинг GET-параметра
gobuster fuzz -u "http://target.com/page.php?id=FUZZ" -w ids.txt

# Фаззинг заголовка
gobuster fuzz -u http://target.com -H "X-Forwarded-For: FUZZ" -w ips.txt

# Фаззинг cookie
gobuster fuzz -u http://target.com -c "session=FUZZ" -w sessions.txt
```

---

## 📌 10. Общие флаги

Эти флаги работают почти во всех режимах:

| Флаг | Описание |
|------|----------|
| `-t <N>` | Число потоков |
| `-o <file>` | Сохранить результат |
| `-q` | Тихий режим (без баннера и прогресса) |
| `-v` | Подробный вывод |
| `--delay <time>` | Задержка между запросами (например, `500ms`) |
| `--timeout <sec>` | Таймаут запроса |
| `--no-error` | Не показывать ошибки |
| `--no-progress` | Не показывать прогресс |
| `-z` | Не показывать прогресс-бар |
| `--help` | Справка по режиму |

---

## 📌 11. Готовые команды для пентеста

### Этап 1: Первичная разведка
```bash
whatweb http://target.com
curl -I http://target.com
curl http://target.com/robots.txt
curl http://target.com/sitemap.xml
```

### Этап 2: Базовый брутфорс директорий
```bash
gobuster dir \
  -u http://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -t 50 \
  -o gobuster_basic.txt
```

### Этап 3: Поиск файлов с расширениями
```bash
gobuster dir \
  -u http://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,bak,old,zip,tar.gz,sql,env,json,xml \
  -s 200,301,302,403 \
  -t 50 \
  -o gobuster_ext.txt
```

### Этап 4: Админки и чувствительные пути
```bash
gobuster dir \
  -u http://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/Admin-Panels.txt \
  -t 30 \
  -o gobuster_admin.txt
```

### Этап 5: Vhost
```bash
gobuster vhost \
  -u http://10.10.10.50 \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain \
  -t 50 \
  -o gobuster_vhost.txt
```

### Этап 6: DNS
```bash
gobuster dns \
  -d target.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -t 50 -i \
  -o gobuster_dns.txt
```

### Этап 7: Через Burp Suite (для анализа)
```bash
gobuster dir \
  -u http://target.com \
  -w common.txt \
  -p http://127.0.0.1:8080 \
  -t 20
```

### Этап 8: Фильтрация soft-404
```bash
# Сначала узнаём длину «заглушки»
curl -s http://target.com/thisdoesnotexist12345 | wc -c

# Потом исключаем её
gobuster dir \
  -u http://target.com \
  -w common.txt \
  --exclude-length 1234 \
  -t 50
```

---

## 📌 12. Словари (wordlists)

### Установка SecLists
```bash
sudo apt install seclists -y
```

### Основные пути
| Словарь | Путь | Для чего |
|---------|------|----------|
| common.txt | `/usr/share/wordlists/dirb/common.txt` | Базовая разведка (~4600 слов) |
| directory-list-2.3-medium.txt | `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` | Глубокая разведка (~220k) |
| directory-list-2.3-big.txt | `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt` | Максимум (~1.2M) |
| Admin-Panels.txt | `/usr/share/seclists/Discovery/Web-Content/Admin-Panels.txt` | Админки |
| Common-PHP-Filenames.txt | `/usr/share/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt` | PHP-файлы |
| subdomains-top1million-5000.txt | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | Поддомены |
| raft-small-words.txt | `/usr/share/seclists/Discovery/Web-Content/raft-small-words.txt` | Быстрый скан |

---

## 📌 13. Что считать красным флагом

| Находка | Что значит | Следующий шаг |
|---------|-----------|---------------|
| `/admin`, `/administrator` | Панель управления | Проверка на слабые пароли, обход аутентификации |
| `/backup`, `/backups` | Резервные копии | Скачать, искать конфиги, пароли |
| `config.php.bak`, `.env` | Конфиг с credentials | Скачать, найти пароли/ключи |
| `/uploads` | Загрузка файлов | Проверить unrestricted file upload |
| `/api` | API | IDOR, broken auth, mass assignment |
| `/.git/`, `/.svn/` | Репозиторий | Выкачать код через `git-dumper` |
| `/phpmyadmin`, `/pma` | Админка БД | Брутфорс, SQLi |
| `/server-status` | Apache статус | Внутренняя информация |
| `/actuator` | Spring Boot | Утечка конфигов, env |
| `dev.`, `staging.`, `test.` | Тестовые среды | Часто менее защищены |
| `403 Forbidden` | Ресурс есть, но закрыт | Обход через заголовки, path traversal |

---

## 📌 14. Сравнение с ffuf и feroxbuster

| Критерий | Gobuster | ffuf | feroxbuster |
|----------|----------|------|-------------|
| Скорость (req/s) | ~500 | ~1400 | ~950 |
| Рекурсия | Ручная | Ручная | **Автоматическая** |
| Фаззинг параметров | Ограничен | **Лучший** | Нет |
| Wildcard-фильтрация | Базовая | Продвинутая | **Автоматическая** |
| Синтаксис | Простой | Сложный | Средний |
| Кривая обучения | Низкая | Высокая | Средняя |

**Workflow:**
1. **Gobuster** — быстрая первичная разведка.
2. **ffuf** — фаззинг параметров, API, заголовков.
3. **feroxbuster** — глубокая рекурсивная разведка.

---

## 📌 15. Типичные ошибки

| Ошибка | Решение |
|--------|---------|
| Тысячи `200 OK` на всё | `--exclude-length <N>` или `-s 200,301,403` |
| Сервер не отвечает | Проверить `curl -I`, добавить `-k` для TLS |
| WAF блокирует | Сменить `-a` (User-Agent), добавить `--delay` |
| Скан уронил сервер | Уменьшить `-t` до 10–20 |
| Ничего не найдено | Сменить словарь, добавить `-x` с расширениями |
| Ложные срабатывания | Использовать `ffuf` или `feroxbuster` |
| Не терять результаты | Всегда `-o file.txt` |

---

## 📌 16. Полезные ссылки

- [Официальный GitHub Gobuster](https://github.com/OJ/gobuster)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [HackTricks: Web Enumeration](https://book.hacktricks.xyz/)
- [OWASP: Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [HTB Academy: Web Enumeration](https://academy.hackthebox.com/)

---

*Обновлено: 2026-09-14*

---
