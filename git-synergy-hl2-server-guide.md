# Руководство по настройке выделенного сервера Synergy Half-Life 2 на Linux

*Последнее обновление: 15 мая 2018*

## Предварительные требования

1. **Учетная запись Steam с приобретенной копией Half-Life 2** (ep0, 1 или 2)
   * Требуется только один раз во время установки, не требуется для последующего запуска сервера (сервер будет работать анонимно)
2. **Ubuntu 17.10 Server 64-bit**
   * Использовалась в моей настройке, но версия 16.04 LTS, вероятно, тоже подойдет
3. **Около 20 ГБ дискового пространства на каждый эпизод**
4. **Соответствующие правила брандмауэра iptables для разрешения трафика** (подробнее описано ниже)
5. **512 МБ оперативной памяти достаточно, 1 виртуальный процессор**

## Настройка пользователя steam и steamcmd

1. Следующие шаги предполагают, что вы работаете от имени root, но можно добавить префикс sudo при необходимости
2. `useradd -m steam`
3. ```
   apt-get update
   apt-get -y dist-upgrade
   apt-get -y autoremove
   apt-get -y autoclean
   apt-get -y clean
   ```
4. `reboot`
5. `apt-get install lib32gcc1`
6. `su - steam`
7. `mkdir ~/Steam && cd ~/Steam`
8. `curl -O "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz"`
   * Установка через `apt-get install steamcmd` не рекомендуется, используйте последнюю версию
9. `tar -xvf steamcmd_linux.tar.gz`
10. Настройте правила брандмауэра:
    ```
    iptables -A INPUT -p udp --dport 27015 -j ACCEPT
    iptables -A INPUT -p tcp --dport 27015 -j ACCEPT
    ```

## Загрузка Half-Life 2 и Synergy

1. `su - steam` (если вы еще не под этим пользователем)
2. `cd ~steam/Steam/`
3. `bash ./steamcmd.sh login YOUR_USERNAME app_update 17520 app_update 220`
   * Введите свой пароль при запросе, а также код 2FA, если потребуется
   * Добавьте дополнительные `app_update ###`, если хотите установить другие эпизоды (hl2=220, ep1=380, ep2=420, lostCoast=340)
4. `cd ~steam/Steam/steamapps/common/Synergy/synergy/`
5. `vi mount.cache`
   * Добавьте игры, которые хотите подключить (hl2, ep1, ep2)
   * Создайте файл, если он отсутствует
   * Пример содержимого: `hl2,ep1,ep2`
6. `vi motd.txt` (сообщение дня для игроков)
7. Настройте конфигурацию сервера:
   ```
   vi cfg/server.cfg
   ```
   Пример содержимого:
   ```
   hostname "НазваниеВашегоСервераВSynergy"
   rcon_password "НадежныйПароль_учтите-не-шифруется-в-сети"
   sv_password "" // Можно оставить пустым для публичного сервера
   sv_tags "mario,luigi"
   sv_cheats 1
   developer 1
   ```

## Запуск сервера

```
cd ~steam/
nohup sudo -i -u steam /home/steam/Steam/steamapps/common/Synergy/srcds_run -console -game synergy -port 27015 +map d1_trainstation_01 -nocrashdialog -insecure -nohltv &
```

## RCON (удаленная консоль для администрирования сервера с клиента)

Эта функция может не работать должным образом, так как TCP-порт привязывается ко второму сетевому адаптеру вместо INET_ANY (0.0.0.0) или вашего основного IP. Это похоже на ошибку, и я не знаю о каких-либо параметрах конфигурации или обходных путях.

Я просто подключаюсь к серверу через SSH и запускаю локальную программу rcon на Python для редактирования переменных в реальном времени. Или можно разместить любые нужные настройки в файле server.cfg.

1. `apt-get install python3-pip` (важно использовать Python 3)
2. `python3 --version`
3. `su - steam`
4. `pip3 install pysrcds`
5. `python3 rcon.py status`

### Пример скрипта rcon.py

```python
from srcds.rcon import RconConnection
import sys

conn = RconConnection('IP_ВАШЕГО_ВТОРОГО_СЕТЕВОГО_ИНТЕРФЕЙСА', '27015', 'НадежныйПароль_учтите-не-шифруется-в-сети', single_packet_mode=True)
response = conn.exec_command(sys.argv[1])

print(response)
```

## Источники

* [Официальная документация Synergy по выделенным серверам](http://www.synergymod.net/Documents/Dedicated%20Servers)
* [GitHub репозиторий pysrcds](https://github.com/pmrowla/pysrcds)
* [Документация Valve по Source Dedicated Server](https://developer.valvesoftware.com/wiki/Source_Dedicated_Server)
* [Форум SRCDS](http://forums.srcds.com/viewtopic/2311)
