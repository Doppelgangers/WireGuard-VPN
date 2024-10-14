# Установка на сервер wireguard с веб-интерфейсом


## проект Wireguard  UI
https://github.com/ngoduykhanh/wireguard-ui/
### Установка
1. Покупаем vps, vds сервер на Debian, Ubuntu
2. Заходим на сервер
    ```shell
    ssh root@<ip адрес сервера>
    ```
    Вводим пароль
3. Обновляем по
    ```shell
    apt update && apt upgrade -y
    ```
4. Устанавливаем _wireguard_
    ```shell
    apt install wireguard -y
    ```
5. Включаем ip forwarding на сервере
    ```shell
    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
    sysctl -p
    ```
    устанавливаем _iptables_:
    ```shell
    apt install iptables -y
    ```

6. Устанавливаем wireguard-ui
    - создаем каталог `wireguard-ui`  и преходим в него
        ```shell
        mkdir wireguard-ui
        cd wireguard-ui
        ```
    - Качаем tar архив из релизов https://github.com/ngoduykhanh/wireguard-ui/releases
        ```sell
        wget <ссылка на tar.gz файл>
        ```
    - Смотрим имя загруженного файла:
        ```shell
        ls -la
        ```
    - Распаковываем скачанный файл:
        ```shell
        tar -xvf <скачанный архив>
        ```

    - Запускаем wireguard-ui для создания конфигурации
        ```shell
        ./wireguard-ui
        ```
    - Останавливаем _wireguard-ui_
        Нажимаем в консоли `Ctrl+C`

7. Запускаем службу для _wireguard_:
    После запуска web интерфейса у нас создастся файл конфигурации для _wireguard_ по пути `/wtc/wireguard/wg0.conf`. Теперь можно включить и запускать _wireguard_ как сервис, чтобы при перезапуске компьютера работал _wireguard_.
    ```shell
    systemctl enable wg-quick@wg0.service
    ```
    - Создаем службы для перезапуска _wireguard_ в случае изменения конфигурации.
        Выполняем в терминале следующие команды:
        ```shell
        cd /etc/systemd/system/
        cat << EOF > wgui.service
        [Unit]
        Description=Restart WireGuard
        After=network.target

        [Service]
        Type=oneshot
        ExecStart=/usr/bin/systemctl restart wg-quick@wg0.service

        [Install]
        RequiredBy=wgui.path
        EOF
        ```

        ```shell
        cd /etc/systemd/system/
        cat << EOF > wgui.path
        [Unit]
        Description=Watch /etc/wireguard/wg0.conf for changes

        [Path]
        PathModified=/etc/wireguard/wg0.conf

        [Install]
        WantedBy=multi-user.target
        EOF
        ```

        ```shell
        systemctl enable wgui.{path,service}
        systemctl start wgui.{path,service}
        ```

7. Создаём файл для автоматического запуска веб-интерфейса в случае перезапуска компьютера:
    ```shell
    cd /etc/systemd/system/
    cat << EOF > wireguard-ui.service
    [Unit]
    Description=WireGuard UI Service
    After=network.target

    [Service]
    Type=simple
    User=root
    WorkingDirectory=/root/wireguard-ui
    ExecStart=/root/wireguard-ui/wireguard-ui
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target

    EOF
    ```
    ```shell
    systemctl enable wireguard-ui.service
    systemctl start wireguard-ui.service
    ```

9. Заходим в веб интерфейс _wireguard-ui_
    - в браузере вводим `http://<адрес сервера>:5000`
    - вводим логин и пароль `admin` `admin`
    - в настройках __Wiregard Server__:
        - в поле __Post Up Script__ вводим:
        ```shell
        iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
        ```
        - в поле __Post Down Script__ вводим:
        ```shell
        iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
        ```
    , где `eth0` - имя сетевого интерфейса, через который будут подключаться клиенты. Скорее всего это `eth0`, но убедиться
    можно выполнив команду `ip a` на сервере и найдя имя интерфейса
    с внешним ip адресом по которому вы подключаетесь к серверу.
    - Сохраняем __Save__ и применяем конфигурацию __Apply Config__.

10. Добавляем пользователей
    - Добавляете пользователей
    - После добавления пользователей надо перезагрузить _wireguard_ применив конфигурацию - нажав кнопку __Apply Config__ в веб интерфейсе.
    - делимся конфигурационным файлом или qr-кодом с пользователем.

