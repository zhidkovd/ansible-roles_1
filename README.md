======= Подключение к хостам =======

Для подключения к хостам по ssh требуется пара ключей, для создания - команда:

ssh-keygen - пароль указывать не требуется

Далее переносим ключи на хосты с помощью команды:

ssh-copy-id <имя пользователя>@ip-address

Выполнять необходимо из пользователя, под которым будет проводиться работа с Ansible.
Проверка корретности подключения по ключу:

ssh <имя пользователя>@<ip-address>

Указание хостов управляется в файле /etc/ansible/hosts, а именно параметры:

<ИМЯ ХОСТА> ansible_ssh_host=ip-address ansible_user=<имя пользователя>

Пример: ubuntu ansible_ssh_host=172.16.0.2 ansible_user=ubuntu

======= Описание ролей =======

В ролях используются глобальные переменные, которые хранятся в vars/vars.yml, там же имеется разделение переменных, которые используются в разных ролях, чтобы было понятнее.

Общий playbook файл называется .play.yml

Установка пакетов происходит из роли "aptpkg". 
Для того, чтобы изменить пакеты, требуемые для установки на удаленную машину, необходимо добавить, удалить или изменить раздел with_items в файле aptpkg/tasks/main.yml.

Изменение файла ssh_config, на удаленной машине происходит в роли "ssh_change_config".
А именно изменяется строчка "#Port 22" на "Port <номер порта>", номер порта используется из переменной.
Также для удобства эта переменная используется в другой роли, чтобы на удаленной машине SSH порт был идеинтичным.
Изменяется строка "PubkeyAuthentication", а именно его значение "no" на "yes".

Добавление новых портов в ufw - allow, используется роль "ufw_allow_ports". 
Изменить порты для открытия можно в файле ufw_allow_ports/tasks/main.yml в строке "with_items".
Также там используется переменная с ssh портом, которая используется также в "ssh_change_config".

Очистка журнала событий выполняется в роли "clear_journalctl_cron"
В файле clear_journalctl_cron/tasks/main.yml, можно указать когда нужно выполнять команду.
Переменная с командой для очистки журнала находится в глобальных переменных.

Копирование и добавление скрипта в cron происходит в роли "add_scripts_cron".
Там происходит проверка директории /var/scripts на удаленной машине, если при проверке выяснилось, что директории не существует, то ansible создает её.
Также происходит проверка наличия скрипта по названию, если он там существует, то ansible удаляет его и копирует заново, это требуется для того,
чтобы при изменении скрипта он также копировался на хост снова.
Переменные с временем, когда устанавливать скрипт в crontab прописаны в vars/vars.yml.
Если протребуется использовать месяц и день месяца, то раскоментировать строки в файле add_scripts_cron/tasks/main.yml.

Для того, чтобы использовался этот репозиторий в ansible нужно раскомментировать 68 строчку файла ansible.cfg
roles_path    = /etc/ansible/roles

Скачивание deb пакета и установка zabbix-agent происходит из роли zabbix_agent_install.
Также используются переменные из vars/vars.yml, а именно:
- url пакета;
- путь к файлу;
- установка пакета с помощью apt.

