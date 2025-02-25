#### iam.serviceAccounts.user {#sa-user}

Роль `iam.serviceAccounts.user` означает, что у пользователя есть права на использование сервисных аккаунтов.
Эта роль нужна, когда пользователь просит сервис выполнять операции от имени какого-то сервисного аккаунта.

Например, при создании группы виртуальных машин вы указываете ваш сервисный аккаунт, а IAM проверяет, что у вас есть разрешение на использование этого аккаунта.

В роль `iam.serviceAccounts.user` входят следующие разрешения:

* получение списка сервисных аккаунтов;
* получение информации о сервисном аккаунте;
* использование сервисного аккаунта при выполнении операций от его имени.

{% include [roles-editor-includes-permissions](iam/roles-editor-includes-permissions.md) %}