Title: Установка VPN-сервера WireGuard на сервер с Debian 10
Summary: Мануал по установке и настройке WireGuard на сервер с Debian 10, для создания VPN-соединения между сервером и своими устройствами на Linux, Windows и Android.
Date: 2021-04-25
Author: rate1  
Category: Linux
Tags: linux, vpn, приватность
Slug: install-wireguard-on-debian-10
Status: draft

## WireGuard 
**WireGuard** - это протокол связи и софт с открытым исходным кодом, реализующий зашифрованные VPN-сети. Протокол WireGuard передает трафик через UDP. Официальный сайт протокола [WireGuard](https://www.wireguard.com/).  
### Установка WireGuard на сервер с Debian 10
Перечисленные ниже комманды выполняем на сервере Debian. 
#### Включение репозитория Debian Backports
Добавляем Backports репозиторий: 
```
:::bash
sudo sh -c "echo 'deb http://deb.debian.org/debian buster-backports main contrib non-free' > /etc/apt/sources.list.d/buster-backports.list"
```
В результате появится новый файл buster-backports.list или в существующий файл будет дописана строка с нужным источником обновлений для Debian.  
#### Обновление системы на сервере
Выполняем команду для обновления сервера: 
```
:::bash
sudo apt update
```

![Обновление сервера Debian](/images/sudo-update.jpg "Обновление сервера Debian")

Выполним поиск пакета WireGuard, чтобы убедиться, что нужный нам репозиторий подключен: 
```
:::bash
apt search wireguard
```
![Поиск пакета wireguard](/images/search-wireguard.jpg "Поиск пакета WireGuard в репозиториях Debian")

Теперь, когда мы убедились, что нужный нам пакет доступен - можно выполлнить установку. 
#### Установка VPN-сервера WireGuard на сервер Debian
Команда для установки: 
```
:::bash
sudo apt install wireguard 
```

![Установка WireGuard на сервер Debian](/images/install-wireguard.jpg "Установка WireGuard на сервер Debian")

#### Настройка WireGuard-сервера
##### Создание приватного и публичного ключей для WireGuard-сервера
Переходим в директорию WireGuard: 
```
:::bash
sudo -i
sudo cd /etc/wireguard/ 
```
Выполняем установку прав для созданных файлов: 
```
:::bash
umask 077
```
Проверяем, что права остались только у владельца файла (в нашем случае root): 

![Проверка прав на созданные файлы](/images/prava.jpg "Проверка прав на созданные файлы")

Ключи будут создаваться с помощью утилиты wg. Создадим сначала приватный ключ: 
```
:::bash
wg genkey > privatekey
```
В результате в директории /etc/wireguard/ будет создан файл **privatekey**, содержащий приватный ключ. Теперь с помощью приватного ключа можно создать публичный (или открытый) ключ: 
```
:::bash
wg pubkey <privatekey> publickey
```
Публичный ключ будет создан в том же каталоге. 
Генерацию ключей можно выполнить одной командой, с помощью команды **tee**: 
```
:::bash
wg genkey | tee privatekey | wg pubkey > publickey
```
##### Настройка интерфейса для WireGuard
Внутри директории /etc/wireguard/ создадим файл конфигурации **wg0.conf**. Добавим в него следующее содержание: 
```
## Установка WireGuard VPN на Debian с помощью файла wg0.conf ##
[Interface]
## Внутренний IP-адрес VPN-сервера ##
Address = 192.168.77.1/24
 
## Порт VPN-сервера ##
ListenPort = 51194
 
## Приватный ключ VPN-сервера, который хранится в файле /etc/wireguard/privatekey ##
PrivateKey = <вставьте сюда свой privatekey>
 
## Сохранять и обновлять этот конфиг, когда добавятся новые VPN-клиенты ##
SaveConfig = true
```
##### Запуск службы WireGuard
Настраиваем запуск службы wireguard во время загрузки: 
```
:::bash
sudo systemctl enable wg-quick@wg0
```
Стартуем службу: 
```
:::bash
sudo systemctl start wg-quick@wg0
```
###### Проверка службы
Теперь можно получить статус запущенной службы: 
```
:::bash
sudo systemctl status wg-quick@wg0
```
Проверим вывод службы: 
```
:::bash
sudo wg
```
И наконец убедимся, что интерфейс с именем wg0 поднялся в системе: 
```
:::bash
sudo ip a show wg0
```
Должно получиться что-то типа этого: 

![Проверка запущенной службы WireGuard](/images/wireguard-status.jpg "Проверка статуса запущенной службы WireGuard")

###### Ошибка при запуске службы и ее решение
На этапе запуска службы может возникнуть ошибка **RTNETLINK answers: Operation not supported**, если при этом команда: 
```
:::bash
sudo modprobe wireguard
```
выдает что-то типа: "modprobe: FATAL: Module wireguard not found in directory", то нужно выполнить команду: 
```
:::bash
sudo apt install linux-headers-$(uname -r|sed 's/[^-]*-[^-]*-//')
```
После этого пакет wireguard должен нормально запуститься. 

### Установка WireGuard на клиентскую машину с Linux
Процесс установки WireGuard на VPN-клиента под управлением Linux, мало чем отличается от установки на сервер Debian. В качестве примера произведем установку на Manjaro. 
#### Установка пакета WireGuard на VPN-клиентскую машину под управлением Manjaro
Необходимо выполнить команду: 
```
:::bash
sudo pamac install wireguard-tools
```
Различные варианты установки WireGuard для разных клиентов есть на официальном сайте WireGuard в разделе [Installation](https://wireguard.com/install/). 
#### Настройка VPN-клиента
Необходимо также, как и на сервере создать конфиг wg0.conf, сгенерировать ключи и заполнить конфиг данными: 
```
:::bash
sudo sh -c 'umask 077; touch /etc/wireguard/wg0.conf' 
sudo -i
cd /etc/wireguard/
umask 077; wg genkey |tee privatekey | wg pubkey > publickey
```
Проверим, что все три файла создались: 

![Проверяем, что файлы ключей сгенерировались на VPN-клиенте](/images/keys-on-vpn-client.jpg)

#### Редактирование конфигурационного файла wg0.conf на VPN-клиенте
Содержимое конфига должно быть таким: 
```
[Interface]
## Приватный ключ этого VPN-клиента ##
PrivateKey = <вставьте свое значение>

## IP-адрес VPN-клиента ##
Address = 192.168.77.2/24
 
[Peer]
## Публичный ключ VPN-сервера ##
PublicKey = <вставьте свое значение>
 
## Доступные IP-адреса ##
AllowedIPs = 192.168.77.0/24
 
## Публичный IP-адрес и порт VPN-сервера ##
Endpoint = <сервер>:51194
 
## Время жизни соединения ##
PersistentKeepalive = 20
```
#### Запуск службы WireGuard на VPN-клиенте и проверка работоспособности
Выполняем следующие команды: 
```
:::bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo systemctl status wg-quick@wg0
```
### Настройка VPN-сервера для подключения VPN-клиента
Осталось добавить информацию в конфигурационный файл **wg0.conf** VPN-сервера о новом VPN-клиенте. 
Перед внесением изменений, необходимо остановить службу командой: 
```
:::bash
sudo systemctl stop wg-quick@wg0
```
После этого в конфиг нужно добавить: 
```
[Peer]
## Публичный ключ VPN-клиента ##
PublicKey = <значение ключа>
 
## IP-адрес VPN-клиента ##
AllowedIPs = 192.168.77.2/32
```
После сохранения внесенных изменений запускаем службу: 
```
:::bash
sudo systemctl start wg-quick@wg0
```
### Проверка работоспособности VPN-тоннеля
Для проверки созданного VPN-тоннеля, введем команду **ping** на клиентской машине: 
```
:::bash
sudo ping -c 4 192.168.77.1
```
VPN-сервер на Debian пингуется без всяких проблем: 

![Пингуем VPN-сервер для проверки настроек тоннеля](/images/ping-vpn-server.jpg "Проверяем настройку VPN-тоннеля с помощью пинга сервера")

### Установка WireGuard на клиентскую машину с Windows
### Установка WireGuard на клиентский девайс с Android
