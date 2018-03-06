# About Alcolytics

Is an open source platform for a web and product analytics. 
It consists of a set of components: JavaScript tracking client for web applications; 
server-side data collector; services for geo-coding and detecting client device type; 
a new server deployment system.
[Read more](https://alco.readme.io/docs)

Платформа для web и продуктовой аналитики с открытым исходным кодом.
Включает в себя JavaScript трекер для сайта; сервис получения, обогащения,
сохранения и стриминга данных; сервисы гео-кодинга и определения типа клиентского устройства;
систему развертывания нового сервера.
[Подробнее](https://alco.readme.io/docs) 

![Alcolytics sheme](https://raw.githubusercontent.com/alcolytics/alco-tracker/master/docs/alco-scheme.png)

# Alcolytics Bootstrap

Ansible playbook для автоматической установки Alcolytics
Скрипт сделан для Ubuntu 16.04, на других работать не будет.

Подразумевается, что у вас уже имеется сервер под Alcolytics под управлением OC Ubuntu 16.04
и поддомен alco.yourdomain.ru (или какой-либо другой) указывет на этот сервер

# Установка

Установка производится либо с самого сервера либо с другого компьютера.

## С сервера

Устновите минимальные необходимые зависимости

    apt -y update
    
    apt install -y python-minimal python-pip python-netaddr git locales
    
    pip install ansible
    
Если возникает ошибка связанная с локалью, установите локаль и запустите еще раз
        
    echo -e 'LANG=en_US.UTF-8\nLC_ALL=en_US.UTF-8' | sudo tee /etc/default/locale
    locale-gen en_US.UTF-8
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8
 
Загрузка скрипта установки
        
    cd ~
    git clone https://github.com/alcolytics/alco-bootstrap

Генерация ssh ключа

    cd ~
    ssh-keygen -t rsa -b 4096 -f alco -C "your_email@example.com"
    cp alco.pub alco-bootstrap /public_keys
    mv alco* ~/.ssh/
    
Ниже есть блок про создание файла инвентаря, сделайте это. 
Запуск процесса установки
    
    cd ~/alco-bootstrap 
    ansible-playbook alco.yml --connection=local

## С другого компьюетра

Подразумевается что у вас Linix/MacOS.
Установите ansible и python-netaddr при помощи используемого в вашей ОС менеджера пакетов.
Ниже есть блок про создание файла инвентаря, сделайте это.

**Установка необходимых зависимостей Ansible.** 

Дополнительно, при первой установке:

* Если авторизация на сервере по паролю, добавьте ` --ask-pass`
* Если вашему пользователю требуется авторизация для `sudo`, добавьте `--ask-become-pass`
    

    ansible-playbook ansible-requirement.yml


**Установка системы**

Загрузка скрипта установки
        
    cd ~
    git clone https://github.com/alcolytics/alco-bootstrap      

Генерация ssh ключа

    cd ~
    ssh-keygen -t rsa -b 4096 -f alco -C "your_email@example.com"
    cp alco.pub alco-bootstrap/public_keys
    mv alco* ~/.ssh/


    cd ~/alco-bootstrap 

Дополнительно, при первой установке:

* Добавьте `-e "initial=yes"`.
* Если авторизация на сервере по паролю, добавьте ` --ask-pass`
* Если вашему пользователю требуется авторизация для `sudo`, добавьте `--ask-become-pass`


    ansible-playbook alco.yml


## Создание файла инвентаря

Создайте файл `inventory/priv_all`

    [priv_all]
    alcostat ansible_host=<домен трекера>
    
    [priv_all:vars]
    mixpanel_token=<токен mixpanel, если есть>
    tracker_domain=<домен трекера>
    contact_email=<контактная почта>
    
    # Ключ от сервиса mailgun, если есть.
    # Обычным пользователям это не нужно. Используется при поддержке нескольких серверов и требует доп настройки. 
    #mailgun_api_key:
    #mailgun_sender: <Alcolytics Setup <hello@alcolytics.ru> 
        
    [alco:children]
    priv_all


## Установка счетчика на сайт

Обращайтесь к докам AlcoJS:

* Репозиторий: https://github.com/alcolytics/alcojs
* Документации: https://alco.readme.io/docs/js-api
* Актуальный сниппет: https://github.com/alcolytics/alcojs/blob/master/snippet

## Кастомизация конфигурации

##№ Изсключение из гита

Монархически принято решение: `inventory` следует именовать с префиксом priv*, 
дабы не разводить бадак в `.gitignore`. Проще всего использовать имя `private`  

### Конфигурация системы

Полный список параметров можно найти в дефолтах ролей в `roles/*/defaults/*` 
Не изменяйтся основной файл `group_vars/all`, создайте отдельный файл совпадающий с названием группы хостов.
Пример: делаете копию `inventory/all` под названием `inventory/private`, создаете файл конфигурации `group_vars/private`,
все, указанные настройки перезапишут аналоги из `all`. 

### Произвольная структура бд (свои миграции)

Пример миграции находится в `clickhouse_schema/`. Чтобы миграция начала накатяваться при запуске скрипта,
укажите в `group_vars/private` список файлов которые будут загружены. Загрузка происходит лишь единожды,
название миграии будет записано в БД и в дальнейшем будет проигнорировано.

    ch_operations_custom:

      # Чистый SQL запрос
      - type: query
        query: 'SELECT count() FROM events'
    
      # Миграция из локального файла
      - type: local_migration
        file: migration1-test-local-migration.yml
    


## Вопросы и общение

* Канал в telegram https://t.me/alcolytic
* Раздел поддержки в документации https://alco.readme.io/discuss

## License

This software is licensed under the MIT license. See [License File](LICENSE) for more information.

