---
title: "Управление доступом в {{ backup-full-name }} (S3)"
description: "Управление доступом в сервисе, предоставляющим решение для резервного копирования данных в {{ yandex-cloud }} — {{ backup-full-name }}. В разделе описано, на какие ресурсы можно назначить роль, какие роли действуют в сервисе."
---

# Управление доступом в {{ backup-name }}

В этом разделе вы узнаете:

* [на какие ресурсы можно назначить роль](#resources);
* [какие роли действуют в сервисе](#roles-list).

{% include [about-access-management](../../_includes/iam/about-access-management.md) %}

## На какие ресурсы можно назначить роль {#resources}

В консоли {{ yandex-cloud }} или с помощью YC CLI вы можете назначить роль на [облако](../../resource-manager/concepts/resources-hierarchy.md#cloud) или [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder). Назначенные роли будут действовать и на вложенные ресурсы.

## Какие роли действуют в сервисе {#roles-list}

### Сервисные роли {#service-roles}

#### backup.admin {#backup-admin}

{% include notitle [roles-backup-admin](../../_includes/roles-backup-admin.md) %}

#### backup.editor {#backup-editor}

{% include notitle [roles-backup-editor](../../_includes/roles-backup-editor.md) %}

#### backup.viewer {#backup-viewer}

{% include notitle [roles-backup-viewer](../../_includes/roles-backup-viewer.md) %}

### Примитивные роли {#primitive-roles}

{% include [roles-primitive](../../_includes/roles-primitive.md) %}
