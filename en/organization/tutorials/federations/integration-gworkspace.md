# Authentication using Google Workspace

With an [identity federation](../../concepts/add-federation.md), you can use [Google Workspace](https://workspace.google.com/) to authenticate users in an organization.

Authentication setup includes the following steps:

1. [Creating and setting up a SAML application in Google Workspace](#gworkspace-settings).

1. [Creating and setting up a federation in {{ org-full-name }}](#yc-settings).

1. [Setting up Single Sign-On (SSO)](#sso-settings).

1. [Authentication](#test-auth).

## Getting started {#before-you-begin}

To follow the steps described in this section, you will need a subscription to Google Workspace services and a verified domain to set up your SAML application for.

## Creating and setting up a SAML application in Google Workspace {#gworkspace-settings}

### Create a SAML application and download a certificate {#create-app}

A SAML application in Google Workspace acts as an identity provider (IdP). Create a SAML application and download a certificate:

1. Open the [Google Workspace Admin Console](https://admin.google.com/).

1. In the left-hand panel, select **Mobile and web applications**.

1. Click **Add** → **Add a custom SAML app**.

1. Enter the name of the app, select the logo, and click **Continue**.

1. In the **Google IdP information** step, the IdP server data is shown. You will need this data when [setting up a federation in {{ org-full-name }}](#yc-settings).

{% note alert %}

Do not close the page where you create an app in Google Workspace: you will get the required configuration data for the **Service provider information** step in [further steps](#add-link).

{% endnote %}

## Creating and setting up a federation in {{ org-full-name }} {#yc-settings}

### Create a federation {#create-federation}

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to [{{ org-full-name }}]({{ link-org-main }}).

   1. In the left-hand panel, select [{{ ui-key.yacloud_org.pages.federations }}]({{ link-org-federations }}) ![icon-federation](../../../_assets/console-icons/vector-square.svg).

   1. Click **{{ ui-key.yacloud_org.form.federation.action.create }}**.

   1. Give your federation a name. It must be unique within the folder.

   1. You can also add a description, if required.

   1. In the **{{ ui-key.yacloud_org.entity.federation.field.cookieMaxAge }}** field, specify the time before the browser asks the user to re-authenticate.

   1. In the **{{ ui-key.yacloud_org.entity.federation.field.issuer }}** field, enter the link from the **Object ID** field on the Google Workspace **Google IdP information** page. The link should have the following format:

      ```
      https://accounts.google.com/o/saml2?idpid=<SAML_application_ID>
      ```

   1. In the **{{ ui-key.yacloud_org.entity.federation.field.ssoUrl }}** field, paste the link copied from the **SSO URL** field on the Google Workspace **Google IdP information** page. The link should have the following format:

      ```
      https://accounts.google.com/o/saml2/idp?idpid=<SAML_application_ID>
      ```
      {% include [ssourl_protocol](../../../_includes/organization/ssourl_protocol.md) %}

   1. Enable **{{ ui-key.yacloud_org.entity.federation.field.autocreateUsers }}** to add authenticated users to your organization automatically. If you do not enable this option, you will need to [manually add](../../operations/add-account.md#add-user-sso) your federated users.

      {% include [fed-users-note](../../../_includes/organization/fed-users-note.md) %}

   1. {% include [forceauthn-option-enable](../../../_includes/organization/forceauthn-option-enable.md) %}

- CLI {#cli}

   {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

   1. See the description of the create federation command:

      ```
      yc organization-manager federation saml create --help
      ```

   1. Create a federation:

      ```bash
      yc organization-manager federation saml create --name my-federation \
          --organization-id <organization_ID> \
          --auto-create-account-on-login \
          --cookie-max-age 12h \
          --issuer "https://accounts.google.com/o/saml2?idpid=<SAML_application_ID>" \
          --sso-url "https://accounts.google.com/o/saml2/idp?idpid=<SAML_application_ID>" \
          --sso-binding POST \
          --force-authn
      ```

      Where:

      * `--name`: Federation name. It must be unique within the folder.

      * `--organization-id`: Organization ID.

      * `--auto-create-account-on-login`: Flag to enable the automatic creation of new cloud users following authentication on the IdP server.
         This option makes it easier to create users; however, users created this way will not be able to do anything with cloud resources. This does not apply to the resources the `allUsers` or `allAuthenticatedUsers` [system group](../../../iam/concepts/access-control/system-group.md) roles are assigned to.

         If this option is disabled, users who are not added to the organization cannot log in to the management console, even if they authenticate with your server. In this case, you can manage a list of users allowed to use {{ yandex-cloud }} resources.

      * `--cookie-max-age`: Time before the browser asks the user to re-authenticate.

      * `--issuer`: ID of the IdP server to be used for authentication.

         Use the link from the **Object ID** field on the Google Workspace **Google IdP information** page. This is a link in the format:

         ```
         https://accounts.google.com/o/saml2?idpid=<SAML_application_ID>
         ```

      * `--sso-url`: URL to which the browser redirects the user for authentication.

         Use the link from the **SSO URL** field on the Google Workspace **Google IdP information** page. The link should have the following format:

         ```
         https://accounts.google.com/o/saml2/idp?idpid=<SAML_application_ID>
         ```

         {% include [ssourl_protocol](../../../_includes/organization/ssourl_protocol.md) %}

      * `--sso-binding`: Specify the Single Sign-on binding type. Most Identity Providers support the `POST` binding type.

      * {% include [forceauthn-cli-enable](../../../_includes/organization/forceauth-cli-enable.md) %}

- {{ TF }} {#tf}

   {% include [terraform-install](../../../_includes/terraform-install.md) %}

   1. Specify the federation parameters in the configuration file:

      * `name`: Federation name. It must be unique within the folder.
      * `description`: Federation description.
      * `organization_id`: Organization ID.
      * `labels`: Set of key/value label pairs assigned to the federation.
      * `issuer`: IdP server ID to be used for authentication.

         Use the link from the **Object ID** field on the Google Workspace **Google IdP information** page. The link should have the following format:

         ```
         https://accounts.google.com/o/saml2?idpid=<SAML_application_ID>
         ```

      * `sso_binding`: Specify the Single Sign-on binding type. Most Identity Providers support the `POST` binding type.
      * `sso_url`: URL of the page the browser redirects the user to for authentication.

         Use this as the destination when copying the link from the **SSO URL** field on the Google Workspace ** Google IdP information** page. The link should have the following format:

         ```
         https://accounts.google.com/o/saml2/idp?idpid=<SAML_application_ID>
         ```

         {% include [ssourl_protocol](../../../_includes/organization/ssourl_protocol.md) %}

      * `cookie_max_age`: Time, in seconds, before the browser asks the user to re-authenticate. The default value is `8 hours`.
      * `auto_create_account_on_login`: Flag to activate the automatic creation of new cloud users after authenticating on the IdP server.
         This option makes it easier to create users; however, users created this way will not be able to do anything with cloud resources. This does not apply to the resources the `allUsers` or `allAuthenticatedUsers` [system group](../../../iam/concepts/access-control/system-group.md) roles are assigned to.

         If this option is disabled, users who are not added to the organization cannot log in to the management console, even if they authenticate with your server. In this case, you can manage a list of users allowed to use {{ yandex-cloud }} resources.
      * `case_insensitive_name_ids`: Flag that indicates whether usernames are case-insensitive.
         If the option is enabled, the IDs of federated user names will be case-insensitive.
      * `security_settings`: Federation security settings:
         * `encrypted_assertions`: Sign authentication requests.
            If this option is enabled, all authentication requests from {{ yandex-cloud }} will have a digital signature. You need to download and install a {{ yandex-cloud }} certificate.

      Here is an example of the configuration file structure:

      ```
      resource "yandex_organizationmanager_saml_federation" federation {
       name            = "my-federation"
       organization_id = "<organization_ID>"
       auto_create_account_on_login = "true"
       issuer          = "https://accounts.google.com/o/saml2?idpid=<SAML_application_ID>"
       sso_url         = "https://accounts.google.com/o/saml2/idp?idpid=<SAML_application_ID>"
       sso_binding     = "POST"
       security_settings {
          encrypted_assertions = "true"
          }
      }
      ```

   1. Make sure the configuration files are valid.

      1. In the command line, go to the directory where you created the configuration file.
      1. Run a check using this command:

         ```
         $ terraform plan
         ```

      If the configuration is described correctly, the terminal displays the federation parameters. If the configuration contains any errors, {{ TF }} will point them out.

   1. Create a federation.

      1. If the configuration does not contain any errors, run this command:

         ```
         $ terraform apply
         ```

      1. Confirm you want to create a federation.

      This will create a federation in the specified organization. You can check the new federation and its settings in the organization's [{{ ui-key.yacloud_org.pages.federations }}]({{ link-org-federations }}) section.

- API {#api}

   1. Create a file with the request body, e.g., `body.json`:

      ```json
      {
        "name": "my-federation",
        "organizationId": "<organization ID>",
        "autoCreateAccountOnLogin": true,
        "cookieMaxAge":"43200s",
        "issuer": "https://accounts.google.com/o/saml2?idpid=<SAML application ID>",
        "ssoUrl": "https://accounts.google.com/o/saml2/idp?idpid=<SAML application ID>",
        "ssoBinding": "POST",
        "securitySettings": {
          "forceAuthn": true
        }
      }
      ```

      Where:

      * `name`: Federation name. It must be unique within the folder.

      * `organizationId`: Organization ID.

      * `autoCreateAccountOnLogin`: Flag to activate the automatic creation of new cloud users after authenticating on the IdP server.
         This option makes it easier to create users; however, users created this way will not be able to do anything with cloud resources. This does not apply to the resources the `allUsers` or `allAuthenticatedUsers` [system group](../../../iam/concepts/access-control/system-group.md) roles are assigned to.

         If this option is disabled, users who are not added to the organization cannot log in to the management console, even if they authenticate with your server. In this case, you can manage a list of users allowed to use {{ yandex-cloud }}resources.

      * `cookieMaxAge`: Time that must elapse before the browser asks the user to re-authenticate.

      * `issuer`: IdP server ID to be used for authentication.

         Use the link from the **Object ID** field on the Google Workspace **Google IdP information** page. The link should have the following format:

         ```
         https://accounts.google.com/o/saml2?idpid=<SAML app ID>
         ```
      * `ssoUrl`: URL of the page the browser redirects the user to for authentication.

         Use this as the destination when copying the link from the **SSO URL** field on the Google Workspace ** Google IdP information** page. The link should have the following format:

         ```
         https://accounts.google.com/o/saml2/idp?idpid=<SAML app ID>
         ```

         {% include [ssourl_protocol](../../../_includes/organization/ssourl_protocol.md) %}

      * `ssoBinding`: Specify the Single Sign-on binding type. Most Identity Providers support the `POST` binding type.

      * {% include [forceauthn-api-enable](../../../_includes/organization/forceauth-api-enable.md) %}

   1. {% include [include](../../../_includes/iam/create-federation-curl.md) %}

{% endlist %}

### Add certificates {#add-certificate}

While authenticating, the {{ org-name }} service should be able to verify the IdP server certificate. To enable this, download a certificate from the open Google Workspace **Google IdP Information** page and add it to the created federation:

{% list tabs group=instructions %}

- Management console {#console}

   1. In the left-hand panel, select [{{ ui-key.yacloud_org.pages.federations }}]({{ link-org-federations }}) ![icon-federation](../../../_assets/console-icons/vector-square.svg).

   1. Click the name of the federation to add a certificate to.

   1. At the bottom of the page, click **{{ ui-key.yacloud_org.entity.certificate.action.add }}**.

   1. Enter certificate name and description.

   1. Choose how to add a certificate:

      * To add a certificate as a file, click **{{ ui-key.yacloud_portal.component.file-input.button_choose }}** and specify the path to it.

      * To paste the contents of a copied certificate, select the **{{ ui-key.yacloud_org.component.form-file-upload.method.manual }}** method and paste the contents.

   1. Click **{{ ui-key.yacloud_org.actions.add }}**.

- CLI {#cli}

   {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

   1. View a description of the add certificate command:

      ```
      yc organization-manager federation saml certificate create --help
      ```

   1. Add a federation certificate by specifying the certificate file path:

      ```
      yc organization-manager federation saml certificate create --federation-id <federation_ID> \
        --name "my-certificate" \
        --certificate-file certificate.pem
      ```

- API {#api}

   Use the [create](../../api-ref/Certificate/create.md) method for the [Certificate](../../api-ref/Certificate/index.md) resource:

   1. Generate the request body. In the `data` property, specify the contents of the certificate:

      ```json
      {
        "federationId": "<federation_ID>",
        "name": "my-certificate",
        "data": "-----BEGIN CERTIFICATE..."
      }
      ```

   1. Send the add certificate request:

      ```bash
      $ export IAM_TOKEN=CggaAT********
      $ curl -X POST \
          -H "Content-Type: application/json" \
          -H "Authorization: Bearer ${IAM_TOKEN}" \
          -d '@body.json' \
          "https://organization-manager.{{ api-host }}/organization-manager/v1/saml/certificates"
      ```

{% endlist %}

{% note tip %}

To make sure authentication is not interrupted when the certificate expires, we recommend adding multiple certificates to your federation, i.e., the current one and those to be used afterwards. If one certificate turns invalid, {{ yandex-cloud }} will attempt to verify the signature with another one.

{% endnote %}


## Setting up Single Sign-On (SSO) {#sso-settings}

### Specify the redirect URL {#add-link}

Once you have created a federation, complete the creation of the SAML application in Google Workspace:

1. Go back to the SAML app creation page's **Google IdP information** step and click **Continue**.

1. In the **Service provider information** step, specify information about {{ yandex-cloud }} that acts as a service provider:

   * In the **ACS URL** and **Object ID** fields, enter the URL to redirect users to after successful authentication:

      ```
      https://{{ auth-host }}/federations/<federation_ID>
      ```

      {% cut "How to get a federation ID" %}

      {% include [get-federation-id](../../../_includes/organization/get-federation-id.md) %}

      {% endcut %}

   * Enable **Signed Response**.

1. Click **Continue**.

   {% note tip %}

   For a user to be able to contact {{ yandex-cloud }} technical support from the [management console]({{ link-console-support }}), in the **Mapping attributes** step, click **Add new mappings** and set up attribute transmission:
   * **Primary email**.
   * **First name**.
   * **Last name**.

   User attributes supported by {{ org-full-name }} services are listed in [{#T}](#claims-mapping).

   {% endnote %}

1. To complete the creation of the app, click **Ready**.

### Add users {#add-users}

1. On the app page, under **User access**, click **Disabled for everyone**.

1. In the page that opens, select who can authenticate with this identity federation:

   * To enable access for all federation users, select **ON for everyone**.

   * To enable access for an individual organizational unit, select the unit from the list on the left and configure the service status for this unit. The child units inherit access settings from the parent units by default.

1. Click **Save**.

### Mapping user attributes {#claims-mapping}

| User data | Comment | Application Attributes |
------------------- | ----------- | -------------------
| Unique user ID | Required attribute. Using an email address is recommended. | **Name ID** field in service provider settings |
| Last name | Displayed in {{ yandex-cloud }} services.<br> Value length limit: {{ saml-limit-last-name }}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` |
| Name | Displayed in {{ yandex-cloud }} services.<br> Value length limit: {{ saml-limit-first-name }}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` |
| Full name | Displayed in {{ yandex-cloud }} services.<br>Example: John Smith.<br> Value length limit: {{ saml-limit-display-name }}. | Attribute unavailable |
| Email | Used to send notifications from {{ yandex-cloud }} services.<br>Example:&nbsp;`smith@example.com`.<br> Value length limit: {{ saml-limit-email }}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Phone | Used to send notifications from {{ yandex-cloud }} services.<br>Example: +71234567890.<br> Value length limit: {{ saml-limit-phone }}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/mobilephone` |
| Profile image | Displayed in {{ yandex-cloud }} services.<br> Value length limit: {{ saml-limit-thumbnail-photo }}. | Attribute unavailable |

{% note warning %}

The `thumbnailPhoto` attribute value exceeding the length limit is ignored. If the value of a different attribute exceeds the limit, the value part that goes beyond the limit is truncated.

{% endnote %}

> Attribute mapping example:
>
> ![image](../../../_assets/organization/google-saml-mapping.png)

### Add users to your organization {#add-users-to-org}

If you did not enable the **{{ ui-key.yacloud_org.entity.federation.field.autocreateUsers }}** option when [creating a federation](#yc-settings), you will have to add federated users to your organization manually.

To do this, you will need user name IDs. They are returned by the IdP server along with a response confirming successful authentication.

{% include [auto-create-users](../../../_includes/organization/auto-create-users.md) %}

A user can be added by an organization administrator (the `organization-manager.admin` role) or owner (the `organization-manager.organizations.owner` role). To learn how to grant roles to a user, see [Roles](../../security/index.md#admin).

{% list tabs group=instructions %}

- Management console {#console}

   1. [Log in]({{ link-passport }}) as the organization administrator or owner.

   1. Go to [{{ org-full-name }}]({{ link-org-main }}).

   1. In the left-hand panel, select [{{ ui-key.yacloud_org.pages.users }}]({{ link-org-users }}) ![icon-users](../../../_assets/console-icons/person.svg).

   1. In the top-right corner, click ![icon-users](../../../_assets/console-icons/chevron-down.svg) → **{{ ui-key.yacloud_org.page.users.action.add-federated-users }}**.

   1. Select the identity federation to add users from.

   1. List the name IDs of users, separating them with line breaks.

   1. Click **{{ ui-key.yacloud_org.actions.add }}**. This will give the users access to the organization.

- CLI {#cli}

   {% include [cli-install](../../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

   1. View a description of the add user command:

      ```
      yc organization-manager federation saml add-user-accounts --help
      ```

   1. Add users by listing their name IDs separated by a comma:

      ```
      yc organization-manager federation saml add-user-accounts --id <federation_ID> \
        --name-ids=alice@example.com,bob@example.com,charlie@example.com
      ```

      Where:

      * `--id`: Federation ID.

      * `--name-ids`: Users' name IDs.

- API {#api}

   To add identity federation users to the cloud:

   1. Create a file with the request body, e.g., `body.json`. In the request body, specify the array of name IDs of users you want to add:

      ```json
      {
        "nameIds": [
          "alice@example.com",
          "bob@example.com",
          "charlie@example.com"
        ]
      }
      ```
   1. Send the request by specifying the Federation ID in the parameters:

      ```bash
      $ curl -X POST \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer <IAM token>" \
        -d '@body.json' \
        https://organization-manager.{{ api-host }}/organization-manager/v1/saml/federations/<federation ID>:addUserAccounts
      ```

{% endlist %}

## Authentication {#test-auth}

When you finish configuring the server, test that everything works properly:

1. Open your browser in guest or private browsing mode.

1. Follow the URL to log in to the management console:

   ```
   https://{{ console-host }}/federations/<federation_ID>
   ```

   {% cut "How to get a federation ID" %}

   {% include [get-federation-id](../../../_includes/organization/get-federation-id.md) %}

   {% endcut %}

   The browser will forward you to the Google authentication page.

1. Enter your credentials and click **Sign in**.

On successful authentication, the IdP server will redirect you back to the `https://{{ auth-host }}/federations/<federation_ID>` URL that you specified in the Google Workspace settings, and then to the [management console]({{ link-console-main }}) home page. In the top-right corner, you can see that you are logged in to the console as a federated user.

#### What's next {#what-is-next}

* [Assign roles to the new users](../../security/index.md#add-role).
