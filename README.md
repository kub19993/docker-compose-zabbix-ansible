# Docker Compose Zabbix + Ansible

Этот репозиторий содержит готовый шаблон для быстрого развертывания системы мониторинга **Zabbix** в Docker и автоматической установки **Zabbix Agent** на целевые серверы с помощью **Ansible**.


## Стек технологий
*   **Zabbix 7.4** (Server, Web-интерфейс, MySQL) в Docker-контейнерах.
*   **Docker Compose** для управления многоконтейнерным приложением.
*   **Ansible** для конфигурирования удаленных хостов (установка и настройка агентов).
*   **Ubuntu Server** (в качестве хост-системы и целевых машин).

---

## Требования для запуска
1.  **Основной сервер (Host)**: Ubuntu Server с установленными Docker, Docker Compose и Ansible.
2.  **Целевой сервер (Target)**: Ubuntu Server (или любая другая ОС с SSH), на который нужно установить Zabbix Agent. Доступ по SSH с основного сервера должен быть настроен.
3.  **Сеть**: Оба сервера должны «видеть» друг друга в сети.

---

## Пошаговая инструкция

### Часть 1: Развертывание Zabbix Server в Docker

1.  **Клонируйте репозиторий** на основной сервер:
    ```bash
    git clone https://github.com/ВАШ_ЛОГИН/docker-compose-zabbix-ansible.git
    cd docker-compose-zabbix-ansible
Запустите контейнеры:

bash
docker compose up -d
Дождитесь инициализации (1–2 минуты). При первом запуске MySQL создает базу, а Zabbix Server — схему. Если Zabbix падает с ошибкой MySQL connection failed, просто перезапустите его:


docker compose restart zabbix-server-mysql
Проверьте работу: Откройте в браузере http://IP_ОСНОВНОГО_СЕРВЕРА:8080.

Логин: Admin

Пароль: zabbix

Часть 2: Установка агента через Ansible
Настройте инвентарь: Откройте файл inventory.ini и укажите IP-адрес целевого сервера, логин и пароль пользователя для SSH-подключения.

ini
[zabbix_agents]
192.168.0.107 ansible_user=zaki ansible_ssh_pass=ваш_пароль ansible_become_pass=ваш_пароль ansible_python_interpreter=/usr/bin/python3
Проверьте плейбук: Откройте install_agent.yml и убедитесь, что в переменной zabbix_server_ip указан IP вашего основного сервера (где запущен Docker).

yaml
vars:
  zabbix_server_ip: "192.168.0.102" # IP сервера с Zabbix
  zabbix_hostname: "ubuntu-server-107"
Запустите плейбук:

bash
ansible-playbook -i inventory.ini install_agent.yml
Часть 3: Добавление хоста в веб-интерфейсе Zabbix
После того как Ansible успешно установит агент, нужно добавить хост в Zabbix, чтобы он начал собирать данные.

	
Зайдите в веб-интерфейс Zabbix.

Перейдите в Data collection -> Hosts.

Нажмите Create host (справа вверху).

Заполните поля:

Host name: Введите имя, которое вы указали в zabbix_hostname (например, ubuntu-server-107).

Templates: Начните вводить Linux by Zabbix agent и выберите его.

Host groups: Выберите Linux servers.

Interfaces: Нажмите Add -> Agent.

IP address: IP вашего целевого сервера (например, 192.168.0.107).

Port: 10050.

Нажмите Add внизу страницы.

Через минуту в списке хостов появится зеленая иконка ZBX, означающая, что мониторинг работает.
