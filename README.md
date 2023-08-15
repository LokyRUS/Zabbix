# Система мониторинга Zabbix
# Установка Zabbix Server с веб-интерфейсом + PostgreSQL.

## Подготовка:
`Создаем виртуальную машину на базе `Debian`:` 
- vm1 user:smirnov = Адаптер1: "Мост"; enp0s3 = ip 192.168.0.150/24

`Обновляем списки пакетов и устанавливаем все обновления данных пакетов:` 

```console
apt update 
```
```console
apt upgrade
```
## Установим `Postgresql` 
- Перед установкой `zabbix server` нудно установить `Postgresql`
```console
apt install postgresql
```
## Установим `zabbix server`
- Для установки, можно воспользоваться офсайтом https://www.zabbix.com/ или использовать команды ниже: 
## установим ZABBIX VERSION 6.0 LTS>>Debian>>11 (Bullseye)>>Server, Frontend, Agent>>PostgreSQL>>Apache
- Скачиваем deb пакет из забикс репазитория
```console
wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-4+debian11_all.deb
```
- устанавливаем его 
```console
dpkg -i zabbix-release_6.0-4+debian11_all.deb
```
- обновляем `apt`
```console
apt update
```
- установим сервер но БЕЗ агента, агента поставим позже
```console
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts
```
- проверям стату и видим что сервер еще не запущен
```console
systemctl status zabbix-server.service
```
## Для того, чтобы `zabbix` мог работать с `Postgresql`, нужно создать пользователя zabbix и базу данных для него, в БД `Postgresql`
- создаем пользователя 
```console
su - postgres -c 'psql --command "CREATE USER zabbix WITH PASSWORD '\'123456789\'';"'
```
- `zabbix`- пользователь;
- `пароль` - 123456789.

- создаем базу данных 
```console
su - postgres -c 'psql --command "CREATE DATABASE zabbix OWNER zabbix;"'
```
## Запускаем скрипт для заполнения бд 
```console
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```
## пропишем Pass для базы данных по пути `/etc/zabbix/zabbix_server.conf`
```console
nano /etc/zabbix/zabbix_server.conf
```
- Находим закомментированную строку 
```console
#DBPassword=password
```
- Раскомментируем и прописываем наш пас 
```console
DBPassword=123456789
```

## перезапускаем `zabbix-server` и `apache2`
```console
systemctl restart zabbix-server apache2
```
- Включаем автозапуск
```console
systemctl enable zabbix-server apache2
```
и проверяем стату 
```console
systemctl status zabbix-server.service
```
Далее смотрим скришоты: 

# ![image1](https://github.com/LokyRUS/Zabbix/blob/sys/images/1.PNG)
- Выбираем тут язык, на котором работет ситема, в данном случае rus

# ![image2](https://github.com/LokyRUS/Zabbix/blob/sys/images/2.PNG)
- Смотрим, нет ли ошибок, если ошибки есть то фиксим

# ![image3](https://github.com/LokyRUS/Zabbix/blob/sys/images/3.PNG)

- На данном этапе, внимально указываем ip сервера или оставляем Localhost, если порт по умочанию то ставляем `0`, указываем пароль и пользователья

# ![image4](https://github.com/LokyRUS/Zabbix/blob/sys/images/4.PNG)

- Если попали на это окно, значит все получилось, указываем имя скрвера и можно запускать.

# ![image5](https://github.com/LokyRUS/Zabbix/blob/sys/images/5.PNG)

Стандртные данные авторизации
- логин- `Admin`
- pass - `zabbix`
## установим `zabbix-agent` на наш хост с `zabbix-server`
```console
apt install zabbix-agent
```
- проверим статус
```console
systemctl status zabbix-agent.service
```
# ![image6](https://github.com/LokyRUS/Zabbix/blob/sys/images/6.PNG)

- Агент подключился

# ![image7](https://github.com/LokyRUS/Zabbix/blob/sys/images/7.PNG)

- полученные метрики 


# Задание 2 

Установите Zabbix Agent на два хоста, один из них наш Zabbix Server.

### Подключим хост к монитроингу 
## Установим `zabbix agent`
- Для установки, можно воспользоваться офсайтом https://www.zabbix.com/ или использовать команды ниже: 
## установим ZABBIX VERSION 6.0 LTS>>Debian>>11 (Bullseye)>>Agent
```console
wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-4+debian11_all.deb
```
- устанавливаем его 
```console
dpkg -i zabbix-release_6.0-4+debian11_all.deb
```
- обновляем `apt`
```console
apt update
```
- установим сервер но БЕЗ агента, агента поставим позже
```console
apt install zabbix-agent
```
- Запускаем 
```console
systemctl restart zabbix-agent
```
- Ставим в автозагрузку
```console
systemctl enable zabbix-agent
```
- проверяем статус 
```console
systemctl status zabbix-agent
```


## Подключаем хост 

# ![image8](https://github.com/LokyRUS/Zabbix/blob/sys/images/8.PNG)
- Настройки>>Узлы cети 
- Жмем создать узел сети

# ![image9](https://github.com/LokyRUS/Zabbix/blob/sys/images/9.PNG)

- Имя узла, любое 
- обязательно назначаем или создаем группу
- добавляем итерфейс `agent` , указываем ip адрес, порт остается по умолчани
- Жмем добавить

# ![image10](https://github.com/LokyRUS/Zabbix/blob/sys/images/10.PNG)

### добавляем метрики для агента 

# ![image11](https://github.com/LokyRUS/Zabbix/blob/sys/images/11.PNG)

- видим что появились метрики, проваливаемся в метрики для принудительного сбора 

# ![image12](https://github.com/LokyRUS/Zabbix/blob/sys/images/12.PNG)

- Выбираем любой метрик и нажимаем `Выполнить сейчас`

# ![image13](https://github.com/LokyRUS/Zabbix/blob/sys/images/13.PNG)

- На данном скире видно, что агент видим но не получил метрики из за отсутствия доступа у серверу

## Решип проблему доступа для агента к серверу 
- зайдем в конф. файл zabbix агета на нашем подключаемом хосте 
```console
nano /etc/zabbix/zabbix_agentd.conf
``` 
Строка 100 
-указываем `ip address` нашего `zabbix-server`, можно как через `,`  так и отдельно
```console
 Server=192.168.0.167
```
## прежде чем перезагрузить агент, посмотрим что с логами, почему не подключился агент к серверу
```
 tail -f /var/log/zabbix-agent/zabbix_agentd.log
``` 
# ![image14](https://github.com/LokyRUS/Zabbix/blob/sys/images/14.PNG)
- Видим, что агент не имеет разрешения на подключение 

## Перезагружаем агент для принятия настроек
```console
systemctl restart zabbix-agent
```
# ![image15](https://github.com/LokyRUS/Zabbix/blob/sys/images/15.PNG)
Видим что агент подключился




