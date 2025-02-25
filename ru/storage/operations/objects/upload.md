# Загрузка объекта

Пользователь может создавать внутри бакета папки и загружать туда объекты. При этом необходимо помнить, что в SDK и HTTP API ключом объекта будет весь путь к объекту от корня бакета. Читайте подробнее в разделе [{#T}](../../concepts/object.md).

{% note info %}

Через консоль управления нельзя загрузить объекты размером более 5 ГБ (см. [{#T}](../../concepts/limits.md)). Также при загрузке через консоль нельзя указать `content-type` и другие заголовки. Для загрузки больших объектов или указания заголовков объекта используйте другие [инструменты](../../tools/index.md).

{% endnote %}


## Простая загрузка {#simple}

{% list tabs group=instructions %}

- Консоль управления {#console}

  Консоль управления позволяет работать с бакетами {{ objstorage-name }} как с иерархической файловой системой.

  Чтобы загрузить объект:
  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в который нужно загрузить объект.
  1. Выберите сервис **{{ objstorage-name }}**.
  1. Нажмите на имя необходимого бакета.
  1. Если вы хотите загрузить объект в конкретную папку, перейдите в нее, нажав на имя. Если хотите создать новую папку, создайте ее, нажав кнопку **{{ ui-key.yacloud.storage.bucket.button_create }}**.
  1. Оказавшись в нужной папке, нажмите кнопку **{{ ui-key.yacloud.storage.bucket.button_upload }}**.
  1. В появившемся окне выберите необходимые файлы и нажмите кнопку **Открыть**.
  1. Консоль управления отобразит все объекты, выбранные для загрузки и предложит для каждого из них выбрать [класс хранилища](../../concepts/storage-class.md). Класс хранилища по умолчанию определяется [настройкой бакета](../../concepts/bucket.md#bucket-settings).
  1. Нажмите кнопку **{{ ui-key.yacloud.storage.button_upload }}**.
  1. Обновите страницу.

  В консоли управления информация о количестве объектов в бакете и занятом месте обновляется с задержкой в несколько минут.

- AWS CLI {#cli}

  1. Если у вас еще нет AWS CLI, [установите и сконфигурируйте его](../../tools/aws-cli.md).
  1. Чтобы загрузить один объект, выполните команду:
 
     ```bash
     aws --endpoint-url=https://{{ s3-storage-host }}/ \
       s3 cp <путь_к_локальному_файлу>/ s3://<имя_бакета>/<ключ_объекта>
     ```
     
     Где:
   
     * `--endpoint-url` — эндпоинт {{ objstorage-name }}.
     * `s3 cp` — команда для загрузки объекта. Чтобы загрузить объект, в первой части команды укажите путь к локальному файлу, который нужно загрузить, а во второй — имя вашего бакета и [ключ](../../concepts/object.md#key), по которому объект будет храниться в бакете.
   
     Чтобы загрузить все объекты из локальной директории, используйте команду:
   
     ```bash
     aws --endpoint-url=https://{{ s3-storage-host }}/ \
       s3 cp --recursive <путь_к_локальной_директории>/ s3://<имя_бакета>/<префикс>/
     ```
   
     Где:
   
     * `--endpoint-url` — эндпоинт {{ objstorage-name }}.
     * `s3 cp --recursive` — команда для загрузки всех объектов из локальной директории, включая вложенные. Чтобы загрузить объекты, в первой части команды укажите путь к папке, из которой нужно скопировать файлы в бакет, а во второй — имя вашего бакета и [идентификатор папки](../../concepts/object.md#folder) в хранилище.

  Команда `aws s3 cp` — высокоуровневая, ее функциональность ограничена. Подробнее см. в [справочнике AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/cp.html). Все возможности загрузки, которые поддерживаются в {{ objstorage-name }}, можно использовать при выполнении команды [aws s3api put-object](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/put-object.html) (см. [ниже](#w-object-lock) примеры работы с [блокировками](../../concepts/object-lock.md)).

- {{ TF }} {#tf}

  {% include [terraform-definition](../../../_tutorials/terraform-definition.md) %}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  Перед началом работы получите [статические ключи доступа](../../../iam/operations/sa/create-access-key.md) — секретный ключ и идентификатор ключа, используемые для аутентификации в {{ objstorage-short-name }}.

  Чтобы создать объект в существующем бакете:

  1. Опишите в конфигурационном файле параметры ресурсов, которые необходимо создать.

     ```hcl
     resource "yandex_storage_object" "test-object" {
       access_key = "<идентификатор_статического_ключа>"
       secret_key = "<секретный_ключ>"
       bucket     = "<имя_бакета>"
       key        = "<имя_объекта>"
       source     = "<путь_к_файлу>"
     }
     ```

     Где:
     * `access_key` — идентификатор статического ключа доступа.
     * `secret_key` — значение секретного ключа доступа.
     * `bucket` — имя бакета для добавления объекта. Обязательный параметр.
     * `key` — имя объекта в бакете. Обязательный параметр. Формат имени:

        {% include [name-format](../../../_includes/name-format.md) %}

     * `source` — относительный или абсолютный путь к файлу, который нужно загрузить в бакет.

     Более подробную информацию о ресурсах, которые вы можете создать с помощью {{ TF }}, см. в [документации провайдера]({{ tf-provider-resources-link }}/storage_object).

  1. Проверьте корректность конфигурационных файлов.

     1. В командной строке перейдите в папку, где вы создали конфигурационный файл.
     1. Выполните проверку с помощью команды:

        ```bash
        terraform plan
        ```

     Если конфигурация описана верно, в терминале отобразится список создаваемых ресурсов и их параметров. Если в конфигурации есть ошибки, {{ TF }} на них укажет. 

  1. Разверните облачные ресурсы.

     1. Если в конфигурации нет ошибок, выполните команду:

        ```bash
        terraform apply
        ```

     1. Подтвердите создание ресурсов: введите в терминал слово `yes` и нажмите **Enter**.

        После этого в указанном каталоге будут созданы все требуемые ресурсы. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}).

- API {#api}

  Чтобы загрузить объект, воспользуйтесь методом S3 API [upload](../../s3/api-ref/object/upload.md).

{% endlist %}


## Загрузка версии объекта с блокировкой (object lock) {#w-object-lock}

Если в бакете включены [версионирование](../buckets/versioning.md) и [блокировки версий объектов](../buckets/configure-object-lock.md), вы можете указать настройки блокировки (запрета на удаление или перезапись) при загрузке версии объекта.

{% list tabs group=instructions %}

- AWS CLI {#cli}

  1. Если у вас еще нет AWS CLI, [установите и сконфигурируйте его](../../tools/aws-cli.md).
  1. Выполните команду:

     ```bash
     aws --endpoint-url=https://{{ s3-storage-host }}/ \
       s3api put-object \
       --body <путь_к_локальному_файлу> \
       --bucket <имя_бакета> \
       --key <ключ_объекта> \
       --object-lock-mode <тип_временной_блокировки> \
       --object-lock-retain-until-date <дата_и_время_окончания_временной_блокировки> \
       --object-lock-legal-hold-status <статус_бессрочной_блокировки>
     ```
     
     Где:
   
     * `--endpoint-url` — эндпоинт {{ objstorage-name }}.
     * `s3api put-object` — команда для загрузки версии объекта. Чтобы загрузить версии объекта с блокировкой, укажите следующие параметры:
       * `--body` — путь к файлу, который нужно загрузить в бакет.
       * `--bucket` — имя вашего бакета.
       * `--key` — [ключ](../../concepts/object.md#key), по которому объект будет храниться в бакете.
       * `--object-lock-mode` — [тип](../../concepts/object-lock.md#types) временной блокировки:
   
         * `GOVERNANCE` — временная управляемая блокировка.
         * `COMPLIANCE` — временная строгая блокировка.
    
       * `--object-lock-retain-until-date` — дата и время окончания временной блокировки в любом из форматов, описанных в [стандарте HTTP](https://www.rfc-editor.org/rfc/rfc9110#name-date-time-formats). Например, `Mon, 12 Dec 2022 09:00:00 GMT`. Указывается только вместе с параметром `--object-lock-mode`.
    
       * `--object-lock-legal-hold-status` — статус [бессрочной блокировки](../../concepts/object-lock.md#types):
    
         * `ON` — блокировка установлена.
         * `OFF` — блокировка не установлена.
    
     Вы можете установить на версию объекта только временную блокировку (параметры `object-lock-mode` и `object-lock-retain-until-date`), только бессрочную блокировку (`object-lock-legal-hold-status`) или обе сразу. Подробнее об их совместной работе см. в разделе [{#T}](../../concepts/object-lock.md#types).

- API {#api}

  Чтобы загрузить версию объекта с блокировкой, воспользуйтесь методом S3 API [upload](../../s3/api-ref/object/upload.md) с заголовками `X-Amz-Object-Lock-Mode` и `X-Amz-Object-Lock-Retain-Until-Date` для временной блокировки и `X-Amz-Object-Lock-Legal-Hold` для бессрочной.

{% endlist %}

Если для бакета также настроены [временные блокировки по умолчанию](../../concepts/object-lock.md#default), все объекты в него нужно загружать с указанием их [MD5-хешей](https://{{ lang }}.wikipedia.org/wiki/MD5):

{% list tabs group=instructions %}

- AWS CLI {#cli}

  1. Вычислите MD5-хэш файла и закодируйте его по схеме [Base64](https://{{ lang }}.wikipedia.org/wiki/Base64):
 
     ```bash
     md5=($(md5sum <путь_к_локальному_файлу>))
     md5_base64=$(echo $md5 | base64)
     ```

  1. Если у вас еще нет AWS CLI, [установите и сконфигурируйте его](../../tools/aws-cli.md).
  1. Загрузите объект в бакет:

     ```bash
     aws --endpoint-url=https://{{ s3-storage-host }}/ \
       s3api put-object \
       --body <путь_к_локальному_файлу> \
       --bucket <имя_бакета> \
       --key <ключ_объекта> \
       --content-md5 $md5_base64
     ```
   
     Где:
   
     * `--endpoint-url` — эндпоинт {{ objstorage-name }}.
     * `s3api put-object` — команда для загрузки версии объекта. Чтобы загрузить версии объекта, укажите следующие параметры:
       * `--body` — путь к файлу, который нужно загрузить в бакет.
       * `--bucket` — имя вашего бакета.
       * `--key` — [ключ](../../concepts/object.md#key), по которому объект будет храниться в бакете.
       * `--content-md5` — закодированный MD5-хеш объекта.
     
     Также вы можете добавить к команде параметры:
     
     * `--object-lock-mode` и `--object-lock-retain-until-date`, чтобы установить на версию объекта временную блокировку, отличную от настроек бакета по умолчанию;
     * `--object-lock-legal-hold-status`, чтобы установить на версию объекта бессрочную блокировку.
 
     Подробнее об этих параметрах см. в инструкции выше.

- API {#api}

  Чтобы загрузить версию объекта с временной блокировкой по умолчанию, воспользуйтесь методом S3 API [upload](../../s3/api-ref/object/upload.md) с заголовком `Content-MD5`.


{% endlist %}
