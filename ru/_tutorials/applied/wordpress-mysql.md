# Сайт на WordPress с кластером БД {{ MY }}

С помощью этой инструкции вы научитесь разворачивать сайт на базе CMS WordPress с [кластером баз данных](../../managed-mysql/concepts/index.md) под управлением СУБД {{ MY }} в инфраструктуре {{ yandex-cloud }}.

Чтобы настроить веб-сайт на WordPress с кластером {{ MY }}:
1. [Подготовьте облако к работе](#before-you-begin).
1. [Создайте виртуальную машину для WordPress](#create-vm).
1. [Создайте кластер БД {{ MY }}](#create-cluster).
1. [Настройте веб-сервер Nginx](#configure-nginx).
1. [Установите WordPress и дополнительные компоненты](#install-wordpress).
1. [Завершите настройку WordPress](#configure-wordpress).
1. [Настройте DNS](#configure-dns).
1. [Проверьте работу веб-сайта](#test-site).

Если сайт вам больше не нужен, [удалите все используемые им ресурсы](#clear-out).

Также инфраструктуру для веб-сайта на WordPress с кластером {{ MY }} можно развернуть через {{ TF }} с помощью [готового файла конфигурации](#terraform).

## Подготовьте облако к работе {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}


### Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки веб-сайта на WordPress с кластером {{ MY }} входит:
* Плата за постоянно запущенную [ВМ](../../compute/concepts/vm.md) (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md)).
* Плата за кластер БД {{ MY }} (см. [тарифы {{ mmy-full-name }}](../../managed-mysql/pricing.md)).
* Плата за использование динамического или статического [внешнего IP-адреса](../../vpc/concepts/address.md#public-addresses) (см. [тарифы {{ vpc-full-name }}](../../vpc/pricing.md)).
* Плата за публичные DNS-запросы и [зоны](../../dns/concepts/dns-zone.md) (см. [тарифы {{ dns-full-name }}](../../dns/pricing.md)).


## Создайте ВМ для WordPress {#create-vm}

Чтобы создать ВМ:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. На странице [каталога](../../resource-manager/concepts/resources-hierarchy.md#folder) в [консоли управления]({{ link-console-main }}) нажмите кнопку **Создать ресурс** и выберите **Виртуальная машина**.
  1. В поле **Имя** введите имя ВМ: `wp-mysql-tutorial-web`.
  1. Выберите [зону доступности](../../overview/concepts/geo-scope.md), в которой будет находиться ВМ.
  1. В блоке **Выбор образа/загрузочного диска** перейдите на вкладку **{{ marketplace-name }}** и выберите публичный образ [Debian 11](/marketplace/products/yc/debian-11), [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts) или [CentOS 7](/marketplace/products/yc/centos-7).
  1. В блоке **Вычислительные ресурсы**:
     * Выберите [платформу](../../compute/concepts/vm-platforms.md).
     * Укажите необходимое количество vCPU и объем RAM:
       * **vCPU** — 2.
       * **Гарантированная доля vCPU** — 20%.
       * **RAM** — 2 ГБ.
  1. В блоке **Сетевые настройки** выберите [сеть](../../vpc/concepts/network.md#network) и [подсеть](../../vpc/concepts/network.md#subnet), к которым нужно подключить ВМ. Если нужной сети или подсети еще нет, вы можете создать их прямо на странице создания ВМ.
  1. В поле **Публичный адрес** оставьте значение **Автоматически**, чтобы назначить ВМ случайный внешний IP-адрес из пула {{ yandex-cloud }}, или выберите статический адрес из списка, если вы зарезервировали его заранее.
  1. Укажите данные для доступа на ВМ:
     * В поле **Логин** введите имя пользователя, например, `yc-user`.
     * В поле **SSH-ключ** вставьте содержимое файла открытого ключа.

       Пару ключей для подключения по [SSH](../../glossary/ssh-keygen.md) необходимо создать самостоятельно, см. [раздел о подключении к ВМ по SSH](../../compute/operations/vm-connect/ssh.md).

     {% note alert %}

     IP-адрес и [имя хоста (FQDN)](../../compute/concepts/network.md#hostname) для подключения к ВМ назначатся ей при создании. Если вы выбрали вариант **Без адреса** в поле **Публичный адрес**, вы не сможете обращаться к ВМ из интернета.

     {% endnote %}

  1. Нажмите кнопку **Создать ВМ**.

- {{ TF }} {#tf}

  См. раздел [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% endlist %}

Создание ВМ может занять несколько минут. Когда ВМ перейдет в [статус](../../compute/concepts/vm-statuses.md) `RUNNING`, вы можете переходить к следующему шагу.

При создании ВМ назначается публичный IP-адрес и имя хоста (FQDN). Эти данные можно использовать для доступа по SSH.

## Создайте кластер БД {{ MY }} {#create-cluster}

Чтобы создать кластер БД {{ MY }}:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. На странице каталога в [консоли управления]({{ link-console-main }}) нажмите кнопку **Создать ресурс** и выберите **Кластер {{ MY }}**.
  1. В поле **Имя** введите имя кластера: `wp-mysql-tutorial-db-cluster`.
  1. В блоке **Класс хоста** выберите **s2.small**.
  1. В блоке **Размер хранилища** укажите 10 ГБ.
  1. В блоке **База данных**:
     * В поле **Имя БД** введите `wp-mysql-tutorial-db`.
     * В поле **Имя пользователя** введите `wordpress`.
     * В поле **Пароль** введите пароль, который вы будете использовать для доступа к БД.
  1. В блоке **Сетевые настройки**:
     * В списке **Сеть** выберите сеть, к которой будет подключен кластер.

  
  1. В блоке **Хосты** добавьте еще два [хоста](../../managed-mysql/concepts/instance-types.md) в других зонах доступности. При создании хостов не включайте для них **Публичный доступ**.


  1. В блоке **Настройки СУБД** нажмите кнопку **Настроить**.

     В поле **Default authentication plugin** выберите пункт **mysql_native_password** и нажмите кнопку **Сохранить**.
  1. Нажмите кнопку **Создать кластер**.

- {{ TF }} {#tf}

  См. раздел [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% endlist %}

Создание кластера БД может занять несколько минут.

## Настройте веб-сервер Nginx {#configure-nginx}

После того как ВМ `wp-mysql-tutorial-web` перейдет в статус `RUNNING`:
1. В блоке **Сеть** на странице ВМ в [консоли управления]({{ link-console-main }}) найдите публичный IP-адрес ВМ.
1. [Подключитесь](../../compute/operations/vm-connect/ssh.md) к ВМ по протоколу SSH. Для этого можно использовать утилиту `ssh` в Linux и macOS и программу [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) для Windows.

   Рекомендуемый способ аутентификации при подключении по SSH — с помощью пары ключей. Не забудьте настроить использование созданной пары ключей: закрытый ключ должен соответствовать открытому ключу, переданному на ВМ.
1. Установите Nginx, менеджер процессов PHP-FPM и дополнительные пакеты:

   {% list tabs group=operating_system %}

   - Debian/Ubuntu {#ubuntu}

     ```bash
     sudo apt-get update
     sudo apt-get install -y nginx-full php-fpm php-mysql
     sudo systemctl enable nginx
     ```

   - CentOS {#centos}

     ```bash
     sudo yum -y install epel-release
     sudo yum -y install nginx
     sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
     sudo yum -y --enablerepo=remi-php74 install php php-mysql php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt
     sudo yum -y --enablerepo=remi-php74 install php-fpm
     sudo systemctl enable nginx
     sudo systemctl enable php-fpm
     ```

   {% endlist %}

1. Задайте настройки веб-сервера в конфигурационных файлах Nginx:

   {% list tabs group=operating_system %}

   - Debian/Ubuntu {#ubuntu}

     1. Вы можете отредактировать файл с помощью редактора `nano`:

        ```bash
        sudo nano /etc/nginx/sites-available/wordpress
        ```

     1. Приведите файл к виду:

        ```nginx
        server {
          listen 80 default_server;

          root /var/www/wordpress;
          index index.php;

          server_name <DNS-имя_сервера>;

          location / {
            try_files $uri $uri/ =404;
          }

          error_page 404 /404.html;
          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
            root /usr/share/nginx/html;
          }

          location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
          }
        }
        ```

     1. Разрешите запуск вашего сайта:

        ```bash
        sudo rm /etc/nginx/sites-enabled/default
        sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
        ```

   - CentOS {#centos}

     Вы можете отредактировать файлы `nginx.conf` и `wordpress.conf` с помощью редактора `nano`:
     1. Откройте файл `nginx.conf`:

        ```bash
        sudo nano /etc/nginx/nginx.conf
        ```

     1. Приведите файл к виду:

        ```nginx
        user nginx;
        worker_processes auto;
        error_log /var/log/nginx/error.log;
        pid /run/nginx.pid;
        include /usr/share/nginx/modules/*.conf;

        events {
          worker_connections 1024;
        }

        http {
          log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';

          access_log  /var/log/nginx/access.log main;

          sendfile            on;
          tcp_nopush          on;
          tcp_nodelay         on;
          keepalive_timeout   65;
          types_hash_max_size 2048;

          include             /etc/nginx/mime.types;
          default_type        application/octet-stream;

          include /etc/nginx/conf.d/*.conf;
        }
        ```

     1. Откройте файл `wordpress.conf`:

        ```bash
        sudo nano /etc/nginx/conf.d/wordpress.conf
        ```

     1. Приведите файл к виду:

        ```nginx
        server {
          listen 80 default_server;

          root /usr/share/nginx/wordpress/;
          index index.php;

          server_name <DNS-имя_сервера>;

          location / {
            try_files $uri $uri/ =404;
          }

          error_page 404 /404.html;
          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
            root /usr/share/nginx/html;
          }

          location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
          }
        }
        ```

   {% endlist %}

## Установите WordPress и дополнительные компоненты {#install-wordpress}

1. Загрузите и распакуйте последнюю версию WordPress:

   {% list tabs group=operating_system %}

   - Debian/Ubuntu {#ubuntu}

     ```bash
     wget https://wordpress.org/latest.tar.gz
     tar -xzf latest.tar.gz
     mv wordpress/wp-config-sample.php wordpress/wp-config.php
     sudo mv wordpress /var/www/wordpress
     sudo chown -R www-data:www-data /var/www/wordpress
     ```

   - CentOS {#centos}

     ```bash
     curl https://wordpress.org/latest.tar.gz --output latest.tar.gz
     tar -xzf latest.tar.gz
     mv wordpress/wp-config-sample.php wordpress/wp-config.php
     sudo mv wordpress /usr/share/nginx/wordpress
     sudo chown -R nginx:nginx /usr/share/nginx/wordpress/
     ```

     Измените настройки SELinux:

     ```bash
     sudo semanage fcontext -a -t httpd_sys_content_t "/usr/share/nginx/wordpress(/.*)?"
     sudo semanage fcontext -a -t httpd_sys_rw_content_t "/usr/share/nginx/wordpress(/.*)?"
     sudo restorecon -R /usr/share/nginx/wordpress
     sudo setsebool -P httpd_can_network_connect 1
     ```

   {% endlist %}

1. Получите ключи безопасности WordPress:

   ```bash
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Сохраните вывод команды — полученные ключи будут нужны на следующем шаге.
1. Добавьте ключи безопасности в конфигурационный файл WordPress `wp-config.php`. Вы можете отредактировать файл с помощью редактора `nano`:

   {% list tabs group=operating_system %}

   - Debian/Ubuntu {#ubuntu}

     ```bash
     sudo nano /var/www/wordpress/wp-config.php
     ```

   - CentOS {#centos}

     ```bash
     sudo nano /usr/share/nginx/wordpress/wp-config.php
     ```

   {% endlist %}

   Замените блок конфигурации на значения, полученные на предыдущем шаге:

   ```php
   define('AUTH_KEY',         't vz,|............R lZ5]');
   define('SECURE_AUTH_KEY',  '@r&pPD............dK-A%=');
   define('LOGGED_IN_KEY',    '%6TuLl............9>/dNE');
   define('NONCE_KEY',        'DO(u.H............$?ja-e');
   define('AUTH_SALT',        '|G Vo<............Xeb.~y');
   define('SECURE_AUTH_SALT', 'Y5tIYA............7Lxf8J');
   define('LOGGED_IN_SALT',   'gR]>WZ............<>|;YY');
   define('NONCE_SALT',       '=]nQIb............HLT2:9');
   ```

1. Перейдите к блоку конфигурации подключения к кластеру `wp-mysql-tutorial-db-cluster`:

   ```php
   // ** {{ MY }} settings - You can get this info from your web host. ** //
   /** The name of the database for WordPress. */

   define( 'DB_NAME', '<DB_NAME>' );
   /** {{ MY }} database username. */
   define( 'DB_USER', '<DB_USER>' );

   /** {{ MY }} database password. */
   define( 'DB_PASSWORD', '<DB_PASSWORD>' );

   /** {{ MY }} hostname. */
   define( 'DB_HOST', '<DB_HOST>' );
   ```

   Укажите в файле вместо:
   * `<DB_NAME>` — имя БД `wp-mysql-tutorial-db`.
   * `<DB_USER>` — имя пользователя `wordpress`.
   * `<DB_PASSWORD>` — пароль, заданный при [создании кластера БД](#create-cluster).
   * `<DB_HOST>` — имя хоста {{ MY }} вида `XXXX-XXXXXXXXXX.{{ dns-zone }}`.

     Чтобы узнать FQDN хоста {{ MY }}:

	 {% list tabs group=instructions %}

	 - Консоль управления {#console}

	   1. Перейдите на страницу кластера {{ MY }}.
       1. На вкладке **Базы данных** нажмите значок ![image](../../_assets/options.svg).
       1. Выберите **Подключиться**.
       1. Найдите строчку `mysql --host=ХХХХ-ХХХХХХХХХХ.{{ dns-zone }}`, где `ХХХХ-ХХХХХХХХХХ.{{ dns-zone }}` — это FQDN хоста с ролью `MASTER`.

     - CLI {#cli}

       [Получите список хостов](../../managed-mysql/operations/hosts.md#list) и скопируйте `NAME` хоста с ролью `MASTER`:

       ```bash
       yc managed-mysql host list --cluster-name <имя_кластера_{{ MY }}>
       ```

       
       ```text
       +------------------------+----------------------+---------+--------+-------------------+-----------+
       |           NAME         |      CLUSTER ID      |  ROLE   | HEALTH |      ZONE ID      | PUBLIC IP |
       +------------------------+----------------------+---------+--------+-------------------+-----------+
       | rc1a-...{{ dns-zone }} | c9quhb1l32unm1sdn0in | MASTER  | ALIVE  | {{ region-id }}-a | false     |
       | rc1b-...{{ dns-zone }} | c9quhb1l32unm1sdn0in | REPLICA | ALIVE  | {{ region-id }}-b | false     |
       +------------------------+----------------------+---------+--------+-------------------+-----------+
       ```


     {% endlist %}

3. Перезапустите Nginx и PHP-FPM:

   {% list tabs group=operating_system %}

   - Debian/Ubuntu {#ubuntu}

     ```bash
     sudo systemctl restart nginx.service
     sudo systemctl restart php7.4-fpm.service
     ```

   - CentOS {#centos}

     ```bash
     sudo systemctl restart nginx.service
     sudo systemctl restart php-fpm.service
     ```

   {% endlist %}

## Завершите настройку WordPress {#configure-wordpress}

1. В блоке **Сеть** на странице ВМ в [консоли управления]({{ link-console-main }}) найдите публичный IP-адрес ВМ.
1. Перейдите по адресу ВМ в браузере.
1. Выберите язык и нажмите кнопку **Продолжить**.
1. Заполните информацию для доступа к сайту:
   * Укажите любое название сайта, например, `wp-your-project`.
   * Укажите имя пользователя, которое будет использоваться для входа в административную панель, например, `admin`.
   * Укажите пароль, который будет использоваться для входа в административную панель.
   * Укажите вашу электронную почту.
1. Нажмите кнопку **Установить WordPress**.
1. Если установка прошла успешно, нажмите кнопку **Войти**.
1. Войдите на сайт, используя указанные на прошлых шагах имя пользователя и пароль. После этого откроется административная панель, в которой можно приступать к работе с вашим сайтом.

## Настройте DNS {#configure-dns}

Если у вас есть зарегистрированное доменное имя, воспользуйтесь сервисом {{ dns-name }} для управления доменом.

Также настроить DNS можно с помощью {{ TF }}, подробнее см. в разделе [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% include [configure-a-record-and-cname](../../_tutorials/_tutorials_includes/configure-a-record-and-cname.md) %}

## Проверьте работу сайта {#test-site}

Чтобы проверить работу сайта, введите в браузере его IP-адрес или доменное имя:
* `http://<публичный_IP-адрес_ВМ>`.
* `http://www.example.com`.

Для входа в панель управления WordPress используйте адрес `http://www.example.com/wp-admin/`.

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за развернутый сайт:
1. [Удалите зону DNS](../../dns/operations/zone-delete.md).
1. [Удалите кластер {{ MY }}](../../managed-mysql/operations/cluster-delete.md).
1. [Удалите ВМ](../../compute/operations/vm-control/vm-delete.md).

Если вы зарезервировали для ВМ статический публичный IP-адрес, [удалите его](../../vpc/operations/address-delete.md).

## Как создать инфраструктуру с помощью {{ TF }} {#terraform}

{% include [terraform-definition](../terraform-definition.md) %}

Чтобы разместить веб-сайт на WordPress с {{ MY }}-кластером с помощью {{ TF }}:
1. [Установите {{ TF }}](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform), [получите данные для аутентификации](../../tutorials/infrastructure-management/terraform-quickstart.md#get-credentials) и укажите источник для установки провайдера {{ yandex-cloud }} (раздел [{#T}](../../tutorials/infrastructure-management/terraform-quickstart.md#configure-provider), шаг 1).
1. Подготовьте файлы с описанием инфраструктуры:

   В руководстве используются [группы безопасности](../../vpc/concepts/security-groups.md). Если они вам недоступны, то запросите доступ в поддержке или уберите из файла конфигурации блок `yandex_vpc_security_group` и другие упоминания `security_group`.

   {% list tabs group=infrastructure_description %}

   - Готовый архив {#ready}

     1. Создайте папку для файлов.
     1. Скачайте [архив](https://{{ s3-storage-host }}/www.example.com/doc-files/wordpress-mysql.zip) (1 КБ).
     1. Разархивируйте архив в папку. В результате в ней должен появиться конфигурационный файл `wordpress-mysql.tf`.

   - Вручную {#manual}

     1. Создайте папку для файлов.
     1. Создайте в папке конфигурационный файл `wordpress-mysql.tf`:

        {% cut "wordpress-mysql.tf" %}

        {% include [wordpress-mysql-tf-config](../../_includes/web/wordpress-mysql-tf-config.md) %}

        {% endcut %}

   {% endlist %}

   Более подробную информацию о параметрах используемых ресурсов в {{ TF }} см. в документации провайдера:
    * [yandex_compute_instance]({{ tf-provider-resources-link }}/compute_instance)
    * [yandex_mdb_mysql_cluster]({{ tf-provider-resources-link }}/mdb_mysql_cluster)
    * [yandex_mdb_mysql_database]({{ tf-provider-resources-link }}/mdb_mysql_database)
    * [yandex_mdb_mysql_user]({{ tf-provider-resources-link }}/mdb_mysql_user)
    * [yandex_vpc_security_group]({{ tf-provider-resources-link }}/vpc_security_group)
    * [yandex_vpc_network]({{ tf-provider-resources-link }}/vpc_network)
    * [yandex_vpc_subnet]({{ tf-provider-resources-link }}/vpc_subnet)
    * [yandex_dns_zone]({{ tf-provider-resources-link }}/dns_zone)
    * [yandex_dns_recordset]({{ tf-provider-resources-link }}/dns_recordset)
1. В блоке `metadata` укажите [метаданные](../../compute/concepts/vm-metadata.md) для создания ВМ `<имя_пользователя>:<содержимое_SSH-ключа>`. Указанное имя пользователя не играет роли, ключ будет присвоен пользователю, который задан в конфигурации образа. В разных образах это разные пользователи. Подробнее см. в разделе [{#T}](../../compute/concepts/vm-metadata.md#keys-processed-in-public-images).
1. В блоке `boot_disk` укажите идентификатор одного из [образов](../../compute/operations/images-with-pre-installed-software/get-list.md) ВМ с нужным набором компонентов:
   * [Debian 11](/marketplace/products/yc/debian-11).
   * [Ubuntu 20.04 LTS](/marketplace/products/yc/ubuntu-20-04-lts).
   * [CentOS 7](/marketplace/products/yc/centos-7).
1. Создайте ресурсы:

   {% include [terraform-validate-plan-apply](../terraform-validate-plan-apply.md) %}

1. [Проверьте работу сайта](#test-site).