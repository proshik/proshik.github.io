+++
title = "Initial server setup with Ubuntu"
date = "2018-03-03"
categories = [
    "DevOps"
]
tags = [ 
    "ubuntu", "linux"
]
+++

Создав новый сервер где-нибудь в облаке его необходимо минимально сконфигурить, создать пользователя отличного от `root`, установить ему нужные права, добавить ssh ключи.

<!--more-->

## Создание нового пользователя

Заходим на машинку по ssh под рутом для того, чтобы создать пользователя с огранничинными привелегиями:

```bash
$ ssh root@айпишник_сервака
```

Создаем пользователя **owner**(он же управленец). Необходимо будет ввести пароль для учетки:

```bash
$ adduser имя_пользователя(owner)
```

### Удаление ползователя

Если что, вдруг создали лишнего, пользователя можно влегкую удалить.

Без удаления домешний директории пользователя:

```bash
$ userdel owner
```

С удалением домашней директории пользователя:

```bash
$ userdel -r owner
```

## Задать привелегии пользователя root

Это нужно для временного получения прав пользователя **root**. Для этого добавим, созданного ранее пользователя в группу **sudo**:

```bash
$ usermod -aG sudo owner
```

## Настройка авторизации по открытому ключу

Сначала необходимо создать ключи для доступа по ним на сервер.

### Создание пары ключей

Выполните команду на **локальной машинке**:

```bash
$ ssh-keygen
```

В результате этого, в поддиректории .ssh домашней директории пользователя будет создан закрытый ключ id_rsa и открытый ключ id_rsa.pub

### Копирование открытого ключа

Необходимо скопировать созданный открытый ключ на ранее созднный сервер

#### Способ через утилиту ssh-copy-id

Запустите скрипт **ssh-copy-id**, указав имя пользователя и айпишник сервера, на который вы хотите установить ключ:

```bash
$ ssh-copy-id owner@айпишник_вашего_сервера
```

После того, как вы введёте пароль, ваш открытый ключ будет добавлен в файл .ssh/authorized_keys на вашем сервере. Соответствующий закрытый ключ теперь может быть использован для входа на сервер.

#### Способ "ручками"

Вводим в консоли команду, чтобы там-же напечатать открытый ключ ```id_rsa.pub```

```bash
$ cat ~/.ssh/id_rsa.pub
```

Нужно выделить открытый ключ и скопировать его в буфер обмена.

Чтобы сделать возможным использование SSH-ключа для авторизации с учетной записью нового удалённого пользователя (remote user), вам необходимо добавить открытый ключ в специальный файл в домашней директории этого пользователя.

На сервере, осуществив вход с учетной записью root-пользователя, выполните следующие команды для переключения на нового пользователя (замените demo на ваше имя пользователя):

```bash
$ su - owner
```

Находясь в директории пользователя создайте новую директорию под названием .ssh и ограничьте права на доступ к ней при помощи следующих команд:

```bash
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
```

Теперь открываем файл в директории .ssh с названием authorized_keys в текстовом редакторе. Мы будем использовать редактор **nano** для этого:

```
$ nano ~/.ssh/authorized_keys
```

Теперь ограничьте права на доступ к файлу authorized_keys при помощи следующей команды:

```bash
$ chmod 600 ~/.ssh/authorized_keys
```

Возвращаемся к пользователю root.

```bash
$ exit
```

Теперь вы можете заходить на сервер по SSH с учетной записью вашего нового пользователя, используя закрытый ключ для авторизации.

## Отключение входа по паролю

В результате этого осуществлять доступ к серверу по SSH можно будет только с использованием вашего открытого ключа. **Его не терять!**
 
Начните с открытия конфигурационного файла в текстовом редакторе под пользователем root или пользователем с правами sudo:

```bash
$ sudo nano /etc/ssh/sshd_config
```

Найдите строку с PasswordAuthentication, раскомментируйте её, удалив # в начале строки, затем измените значение на no. Теперь строка должна выглядеть вот так:

```bash
$ PasswordAuthentication no
```

В этом же файле две другие настройки, необходимые для отключения аутентификации по паролю, уже имеют корректные настройки по умолчанию, не изменяйте эти настройки, если вы ранее не изменяли этот файл:

```bash
$ PubkeyAuthentication yes
$ ChallengeResponseAuthentication no
```

После внесения изменений в этот файл, сохраните и закройте его (CTRL-X, затем Y, далее ENTER)

Перезапустите демон SSH:

```bash
$ sudo systemctl reload sshd
```

Теперь аутентификация по паролю отключена. Ваш сервер доступен для входа через SSH только с помощью ключа.

*На этом все!*