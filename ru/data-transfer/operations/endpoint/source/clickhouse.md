---
title: "Как настроить эндпоинт-источник {{ CH }} в {{ data-transfer-full-name }}"
description: "Из статьи вы узнаете, как задать настройки при создании или изменении эндпоинта-источника {{ CH }} в {{ data-transfer-full-name }}."
---

# Передача данных из эндпоинта-источника {{ CH }}

С помощью сервиса {{ data-transfer-full-name }} вы можете переносить данные из базы {{ CH }} и реализовывать различные сценарии переноса, обработки и трансформации данных. Для реализации трансфера:

1. [Ознакомьтесь с возможными сценариями передачи данных](#scenarios).
1. [Подготовьте базу данных {{ CH }}](#prepare) к трансферу.
1. [Настройте эндпоинт-источник](#endpoint-settings) в {{ data-transfer-full-name }}.
1. [Настройте один из поддерживаемых приемников данных](#supported-targets).
1. [Cоздайте](../../transfer.md#create) и [запустите](../../transfer.md#activate) трансфер.
1. [Выполняйте необходимые действия по работе с базой](#db-actions) и [контролируйте трансфер](../../monitoring.md).
1. При возникновении проблем, [воспользуйтесь готовыми решениями](#troubleshooting) по их устранению.

## Сценарии передачи данных из {{ CH }} {#scenarios}

{% include [migration](../../../../_includes/data-transfer/scenario-captions/migration.md) %}

* [Миграция кластера {{ CH }}](../../../tutorials/managed-clickhouse.md).

Подробное описание возможных сценариев передачи данных в {{ data-transfer-full-name }} см. в разделе [Практические руководства](../../../tutorials/index.md).

## Подготовка базы данных источника {#prepare}

{% include [prepare clickhouse_db](../../../../_includes/data-transfer/endpoints/sources/clickhouse-prepare.md) %}

## Настройка эндпоинта-источника {{ CH }} {#endpoint-settings}

При [создании](../index.md#create) или [изменении](../index.md#update) эндпоинта вы можете задать:

* Настройки подключения к [кластеру {{ mch-full-name }}](#managed-service) или [пользовательской инсталляции](#on-premise), в т. ч. на базе виртуальных машин {{ compute-full-name }}. Эти параметры обязательные.
* [Дополнительные параметры](#additional-settings).

### Кластер {{ mch-name }} {#managed-service}


{% note warning %}

Для создания или редактирования эндпоинта управляемой базы данных вам потребуется [роль `{{ roles.mch.viewer }}`](../../../../managed-clickhouse/security/index.md#mch-viewer) или примитивная [роль `viewer`](../../../../iam/concepts/access-control/roles.md#viewer), выданная на каталог кластера этой управляемой базы данных.

{% endnote %}


Подключение к БД с указанием идентификатора кластера в {{ yandex-cloud }}.

{% list tabs group=instructions %}

- Консоль управления {#console}

    {% include [Managed ClickHouse UI](../../../../_includes/data-transfer/necessary-settings/ui/managed-clickhouse.md) %}

- CLI {#cli}

    * Тип эндпоинта — `clickhouse-source`.

    {% include [Managed ClickHouse CLI](../../../../_includes/data-transfer/necessary-settings/cli/managed-clickhouse.md) %}

- {{ TF }} {#tf}

    * Тип эндпоинта — `clickhouse_source`.

    {% include [Managed ClickHouse {{ TF }}](../../../../_includes/data-transfer/necessary-settings/terraform/managed-clickhouse.md) %}

    Пример структуры конфигурационного файла:

    
    ```hcl
    resource "yandex_datatransfer_endpoint" "<имя_эндпоинта_в_{{ TF }}>" {
      name = "<имя_эндпоинта>"
      settings {
        clickhouse_source {
          security_groups = ["<список_идентификаторов_групп_безопасности>"]
          subnet_id       = "<идентификатор_подсети>"
          connection {
            connection_options {
              mdb_cluster_id = "<идентификатор_кластера>"
              database       = "<имя_переносимой_базы_данных>"
              user           = "<имя_пользователя_для_подключения>"
              password {
                raw = "<пароль_пользователя>"
              }
            }
          }
          <дополнительные_настройки_эндпоинта>
        }
      }
    }
    ```


    Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-dt-endpoint }}).

- API {#api}

    {% include [Managed ClickHouse API](../../../../_includes/data-transfer/necessary-settings/api/managed-clickhouse.md) %}

{% endlist %}

### Пользовательская инсталляция {#on-premise}

Подключение к БД с явным указанием сетевых адресов и портов.

{% list tabs group=instructions %}

- Консоль управления {#console}

    {% include [On premise ClickHouse UI](../../../../_includes/data-transfer/necessary-settings/ui/on-premise-clickhouse.md) %}

- CLI {#cli}

    * Тип эндпоинта — `clickhouse-source`.

    {% include [On premise ClickHouse CLI](../../../../_includes/data-transfer/necessary-settings/cli/on-premise-clickhouse.md) %}

- {{ TF }} {#tf}

    * Тип эндпоинта — `clickhouse_source`.

    {% include [On premise ClickHouse {{ TF }}](../../../../_includes/data-transfer/necessary-settings/terraform/on-premise-clickhouse.md) %}

    Пример структуры конфигурационного файла:

    
    ```hcl
    resource "yandex_datatransfer_endpoint" "<имя_эндпоинта_в_{{ TF }}>" {
      name = "<имя_эндпоинта>"
      settings {
        clickhouse_source {
          security_groups = ["<список_идентификаторов_групп_безопасности>"]
          subnet_id       = "<идентификатор_подсети>"
          connection {
            connection_options {
              on_premise {
                http_port   = "<порт_для_подключения_по_HTTP>"
                native_port = "<порт_для_подключения_к_нативному_интерфейсу>"
                shards {
                  name  = "<имя_шарда>"
                  hosts = [ "список IP-адресов или FQDN хостов шарда" ]
                }
                tls_mode {
                  enabled {
                    ca_certificate = "<сертификат_в_формате_PEM>"
                  }
                }
              }
              database = "<имя_переносимой_базы_данных>"
              user     = "<имя_пользователя_для_подключения>"
              password {
                raw = "<пароль_пользователя>"
              }
            }
          }
          <дополнительные_настройки_эндпоинта>
        }
      }
    }
    ```


    Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-dt-endpoint }}).

- API {#api}

    {% include [On premise ClickHouse API](../../../../_includes/data-transfer/necessary-settings/api/on-premise-clickhouse.md) %}

{% endlist %}

### Дополнительные настройки {#additional-settings}

{% list tabs group=instructions %}

- Консоль управления {#console}

    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.clickhouse.console.form.clickhouse.ClickHouseTableFilter.include_tables.title }}** — будут передаваться данные только из таблиц этого списка.

        {% include [Description for Included tables](../../../../_includes/data-transfer/fields/description-included-tables.md) %}

    * **{{ ui-key.yc-data-transfer.data-transfer.console.form.clickhouse.console.form.clickhouse.ClickHouseTableFilter.exclude_tables.title }}** — данные таблиц из этого списка передаваться не будут.

    Списки включают имя схемы (описание содержания, структуры и ограничений целостности базы данных) и имя таблицы. Для обоих списков поддерживаются выражения вида:

    * `<имя_схемы>.<имя_таблицы>` — полное имя таблицы;
    * `<имя_схемы>.*` — все таблицы в указанной схеме;
    * `<имя_таблицы>` — таблица в схеме по умолчанию.

    Оставьте списки пустыми для переноса всех таблиц.

- {{ TF }} {#tf}

    * `include_tables` — список включенных таблиц. Будут передаваться данные только из таблиц этого списка.

        {% include [Description for Included tables](../../../../_includes/data-transfer/fields/description-included-tables.md) %}

    * `exclude_tables` — список исключенных таблиц. Данные таблиц из этого списка передаваться не будут.

  Списки включают имя схемы (описание содержания, структуры и ограничений целостности базы данных) и имя таблицы. Для обоих списков поддерживаются выражения вида:

    * `<имя_схемы>.<имя_таблицы>` — полное имя таблицы;
    * `<имя_схемы>.*` — все таблицы в указанной схеме;
    * `<имя_таблицы>` — таблица в схеме по умолчанию.

{% endlist %}

### Известные ограничения {#known-limitations}

Если таблицы источника {{ CH }} будут содержать колонки следующих типов, трансфер завершится с ошибкой:

| Тип                 | Пример ошибки                                                     |
|---------------------|-------------------------------------------------------------------|
| `Int128`            | `unhandled type Int128`                                           |
| `Int256`            | `unhandled type Int256`                                           |
| `UInt128`           | `unhandled type UInt128`                                          |
| `UInt256`           | `unhandled type UInt256`                                          |
| `Bool`              | `unhandled type Bool`                                             |
| `Date32`            | `unhandled type Date32`                                           |
| `JSON`              | `unhandled type '<имя_поля> <название_типа>'`                     |
| `Array(Date)`       | `Can't transfer type 'Array(Date)', column '<имя_колонки>'`       |
| `Array(DateTime)`   | `Can't transfer type 'Array(DateTime)', column '<имя_колонки>'`   |
| `Array(DateTime64)` | `Can't transfer type 'Array(DateTime64)', column '<имя_колонки>'` |
| `Map(,)`            | `unhandled type Map(<название_типа>, <название_типа>)`            |

#### Поддерживаемые виды таблиц {#known-limitations-table-types}

Если кластер {{ CH }} содержит более одного хоста, трансфер поддерживает перенос таблиц и материализованных представлений только с движками на базе `ReplicatedMergeTree` либо `Distributed`. Также, данные таблицы и представления должны присутствовать на всех хостах кластера.

В случае, если в списке включенных таблиц присутствуют таблицы или представления с другими движками, либо они присутствуют не на всех хостах кластера — трансфер завершится с ошибкой: `the following tables have not Distributed or Replicated engines and are not yet supported`.

#### Имена баз данных {#known-limitations-db-names}

{{ data-transfer-full-name }} не может переносить базы данных {{ CH }}, в названии которых есть дефис.

## Настройка приемника данных {#supported-targets}

Настройте эндпоинт-приемник:

* [{{ CH }}](../target/clickhouse.md).

Полный список поддерживаемых источников и приемников в {{ data-transfer-full-name }} см. в разделе [Доступные трансферы](../../../transfer-matrix.md).

После настройки источника и приемника данных [создайте и запустите трансфер](../../transfer.md#create).

## Решение проблем, возникающих при переносе данных {#troubleshooting}

* [Не добавляются новые таблицы](#no-new-tables)
* [Не переносятся данные](#no-transfer)

См. полный список рекомендаций в разделе [Решение проблем](../../../troubleshooting/index.md).

{% include [no-new-tables](../../../../_includes/data-transfer/troubles/no-new-tables.md) %}

{% include [table-names](../../../../_includes/data-transfer/troubles/table-names.md) %}

{% include [clickhouse-disclaimer](../../../../_includes/clickhouse-disclaimer.md) %}
