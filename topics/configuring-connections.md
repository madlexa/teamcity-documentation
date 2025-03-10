[//]: # (title: Configuring Connections)
[//]: # (auxiliary-id: Configuring Connections)

TeamCity allows storing presets of connections to external services. You can reuse these presets in various places on the server: when creating projects, configuring notifications, integrating with issue trackers, and more. This article gives instructions on how to add each type of connection.

To add a connection, go the target project's settings, open the __Connections__ page, and click __Add Connection__. Select the connection type, set its _Display name_ to distinguish it from the others, and configure it as described below.

When created, a connection can be used in all the nested subprojects of the current project. If you add a connection in the Root project, it will become available on the whole server.

If your TeamCity server is [installed behind a proxy](configuring-proxy-server.md), it is important to ensure that this is reflected in the connection settings, if applicable. When configuring a callback URL for a connection, you need to specify all URLs by which the current server can be accessed.  
After configuring the proxy, remember to also set the new address as the _Server URL_ in __Global Settings__ of TeamCity.
{product="tc"}

## Azure DevOps

<chunk include-id="azure-devops">

There are two types of Azure DevOps connections in TeamCity:
* __Azure DevOps OAuth 2.0__ allows signing in to TeamCity via an Azure DevOps Services account and creating TeamCity projects from Azure Git and TFVC repositories.
* __Azure DevOps PAT__ allows creating TeamCity projects from Azure Git and TFVC repositories.

<anchor name="azure-devops-connection"/>

#### Azure DevOps OAuth 2.0 Connection
{id="Connecting+to+Azure+DevOps" auxiliary-id="Connecting to Azure DevOps"}

This type of connection supports only Azure DevOps Services. It uses the [OAuth 2.0 protocol](https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops) based on JWT tokens and requires creating a dedicated app in your Azure profile.

This connection can be used for [authenticating users via Azure DevOps](configuring-authentication-settings.md#Azure+DevOps+Services) as well as creating projects and build configurations.

To configure an Azure DevOps OAuth 2.0 connection:
1. In __Project Administration | Connections__, click __Add Connection__.
2. Select _Azure DevOps OAuth 2.0_ as the connection type.
3. TeamCity will display the _Callback URL_ and _scopes_ required for registering an OAuth application in Azure DevOps.  
   Go to the [Register Application](https://app.vsaex.visualstudio.com/app/register) page in Azure and create a new app using the provided parameters. When created, copy the app's ID and client secret.
4. Go back to the connection form in TeamCity and enter the Azure DevOps Services URL, the new application ID, and client secret.
5. Specify the application scope that must be the same as the scope of the created Azure DevOps OAuth App.
6. Save the connection.

To activate the Azure DevOps Services authentication on your server, proceed to enabling the respective [authentication module](configuring-authentication-settings.md#Azure+DevOps+Services).

#### Azure DevOps PAT Connection

This type of connection uses personal access tokens. It allows creating a [project from a Git or TFVC repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL), creating an [Azure DevOps VCS root](azure-devops.md), or integrating with the [Azure Board Work Items](azure-board-work-items.md) tracker.

To configure an Azure DevOps PAT connection:
1. In __Project Administration | Connections__, click __Add Connection__.
2. Select _Azure DevOps PAT_ as the connection type.   
   The page that opens provides the parameters to be used when connecting TeamCity to Azure DevOps Services.
3. Log in to your Azure DevOps Services account to create a personal access token with _All scopes_ as described in the [Microsoft documentation](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate).
4. Continue configuring the connection in TeamCity: on the __Add Connection__ page that is open, specify
   * the server URL in the `https://{account}.visualstudio.com` format or your Azure DevOps Server as `https://{server}:8080/tfs/`
   * your personal access token
5. Save the connection settings.
6. The connection is configured, and now a small Azure DevOps Services icon becomes active in several places where a repository URL can be specified: [create project from URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL), [create VCS root from URL](guess-settings-from-repository-url.md), create [Azure DevOps Server](azure-devops.md) VCS root, create [Azure Board Work Items](azure-board-work-items.md) tracker. Click the icon, log in to Azure DevOps Services and authorize TeamCity. TeamCity will be granted full access to all the resources that are available to you.   
   When configuring Commit Status Publisher for Git repositories hosted in TFS/VSTS, the personal access token can be filled out automatically if a VSTS project connection is configured.

>It is possible to configure several VSTS connections. In this case, the server URL will be displayed next to the VSTS icon to distinguish the server in use.

</chunk>

## Bitbucket Cloud

<chunk include-id="bb-cloud">

>In TeamCity Cloud, a connection to Bitbucket Cloud is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{product="tcc"}

A connection to Bitbucket Cloud can be used to:
* Create a [project from Bitbucket URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Create a [Mercurial VCS root](mercurial.md).
* Integrate with a [Bitbucket Cloud issue tracker](bitbucket-cloud.md).
* Enable [BitBucket Cloud authentication](configuring-authentication-settings.md#Bitbucket+Cloud).

The Bitbucket Cloud connection form provides multiple parameters. You need to use them for [creating a new OAuth consumer in Bitbucket](https://confluence.atlassian.com/bitbucket/oauth-on-bitbucket-cloud-238027431.html#OAuthonBitbucketCloud-Createaconsumer).

After the consumer is created:
1. Copy its key and secret.
2. Go back to the connection form in TeamCity.
3. Paste the key and secret.
4. Save the connection.

A Bitbucket icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your Bitbucket profile. TeamCity will be granted access to your repositories. 
If you configure multiple Bitbucket connections, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</chunk>

## GitHub

<chunk include-id="github">

>In TeamCity Cloud, a connection to GitHub.com is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{product="tcc"}

There are two types of GitHub connections: __GitHub Enterprise__ and __GitHub.com__. Choose it depending on your GitHub account type.

A connection to GitHub can be used to:
* Create a [project from GitHub URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Create a [Git VCS root](git.md).
* Integrate with a [GitHub issue tracker](github.md).
* Enable [GitHub.com authentication](configuring-authentication-settings.md#GitHub.com).

The GitHub connection form provides multiple parameters. You need to use them to [create a new OAuth application in GitHub](https://docs.github.com/en/developers/apps/authorizing-oauth-apps).

After the app is created:
1. Copy its client ID and secret.
2. Go back to the connection form in TeamCity.
3. Paste the GitHub server URL (only for Enterprise) and the app ID and secret.
4. Save the connection.

If you use a GitHub Enterprise server with HTTPS, you need to also upload its HTTPS certificate as described [here](uploading-ssl-certificates.md).

>If you enable the [GitHub.com authentication](configuring-authentication-settings.md#GitHub.com) module and want to restrict access to TeamCity to users of specific GitHub organizations, you need to ensure that your OAuth app is allowed by all these organizations. By default, GitHub does not allow OAuth apps to access the organizations. You can either disable this restriction for all apps or approve only the TeamCity app in each of the required organizations. Please refer to the [GitHub documentation](https://docs.github.com/en/github/setting-up-and-managing-organizations-and-teams/about-oauth-app-access-restrictions) for more details.

A GitHub icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your GitHub profile. TeamCity will be granted full control of your private repositories and get the _Write repository hooks_ permission. If you configure multiple GitHub integrations, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</chunk>

## GitLab

<chunk include-id="gitlab">

>In TeamCity Cloud, a connection to GitLab.com is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{product="tcc"}

There are two types of GitLab connections: __GitLab CE/EE__ and __GitLab.com__. Choose it depending on your GitHub account type.

A connection to GitLab can be used to:
* Create a [project from GitLab URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Integrate with a [GitLab issue tracker](gitlab.md).
* Enable [GitLab.com authentication](configuring-authentication-settings.md#GitLab.com).

The GitLab connection form provides multiple parameters. You need to use them to [create a new OAuth application in GitLab](https://docs.gitlab.com/ee/integration/oauth_provider.html).

After the app is created:
1. Copy its client ID and secret.
2. Go back to the connection form in TeamCity.
3. Paste the GitLab server URL (only for CE/EE) and the app ID and secret.
4. Save the connection.

If you use a GitLab CE/EE server with HTTPS, you need to also upload its HTTPS certificate as described [here](uploading-ssl-certificates.md).

A GitLab icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your GitLab profile. TeamCity will be granted access to your repositories. If you configure multiple GitLab connections, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</chunk>

## Google

This type of connection supports Google Services. It uses the [OAuth 2.0 protocol](https://developers.google.com/identity/protocols/oauth2). 

This connection is used for authentication in TeamCity with a Google account.

Before configuring a Google connection, you need to [create a new Google project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) and [register your app](https://developers.google.com/workspace/guides/configure-oauth-consent) if you have not done it before.

To configure a Google connection in TeamCity:
1. In the **Administration** area, select **Root project | Connections**, and click **Add Connection**.
2. Select _Google_ as the connection type.
3. TeamCity will display the redirect URLs required for registering an OAuth client. Copy them.
4. Go to the [Credentials](https://console.cloud.google.com/apis/credentials) page in the Google project and create the [OAuth client ID](https://developers.google.com/workspace/guides/create-credentials#oauth-client-id) with the Web application type.
5. Paste your Callback URLs to the **Authorized redirect URIs** section in Google OAuth client ID. When the OAuth client is created, copy the Client ID and Client Secret.
6. Go back to the connection form in TeamCity, and enter the Client ID and Client secret.
7. Save the connection.

Now you can enable the [Google authentication module](configuring-authentication-settings.md#Google).

## Docker Registry

A connection to Docker Registry can be used to:
* Sign in to an authenticated Docker registry before running a build / sign out after the build.
* Clean up published images after the build.

See more information in the [dedicated article](configuring-connections-to-docker.md).

## Amazon Web Services (AWS)
{id="AmazonWebServices"}

The Amazon Web Services (AWS) connection allows defining AWS credentials once and using them in builds via the [AWS Credentials](aws-credentials.md) build feature. You can use different AWS credential types: access keys, IAM Role, and the Default credential provider chain.

To configure an AWS connection in TeamCity:

1. In **Project Administration | Connections**, click **Add Connection**.
2. Select _Amazon Web Services (AWS)_ as the connection type.
3. Provide a name to distinguish this connection from others.
4. The _Connection ID_ field is filled out automatically. You can modify it and provide your own unique ID.
5. Select the _AWS region_ where the target resources are located.
6. From the _Type_ drop-down, select one of the credential types:

   {collapsible="true" style="full"}
   Access keys
   : 
   If you selected access keys as the credentials type, get the keys from the AWS console's [Identity and Access Management section](https://console.aws.amazon.com/iam) and provide them to TeamCity. See how to get these keys [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).
   
     In the Access keys section in the TeamCity UI, do the following:
     1. Specify permanent **Access keys**:
          * **Access key ID**. Enter the access key ID.
          * **Secret access Key**. Enter the secret access key.

        It is [recommended](https://aws.amazon.com/blogs/security/how-to-rotate-access-keys-for-iam-users/) to change access keys regularly for security reasons. You will be able to do this after the connection is created via the **Rotate key** button.

        TeamCity will not revoke old keys immediately. After a new key is generated, TeamCity will preserve the old inactive key for 24 hours and then remove it. The lifetime of old keys can be changed via the following properties: `teamcity.internal.cloud.aws.keyRotation.old.key.preserve.time.min` or `teamcity.internal.cloud.aws.keyRotation.old.key.preserve.time.days`.
     
     2. Configure temporary **Session Settings**:
          * **Use session credentials**. Check the box to use an endpoint that provides [temporary access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) keys via AWS STS. Such credentials are short-term (the default session duration is 60 minutes). You can override the default session duration in the [AWS Credentials build feature](https://docs.google.com/document/d/1Y1eJCpErG8NJ9RHPvPOnfZUZZrVRkP8I85BXXSuNYEg/edit#). These credentials do not belong to a specific user and can be provided on demand to grant temporary access to specific resources. We recommend using temporary credentials since they provide better security.
          * **STS endpoint**.
            * TeamCity generates this field automatically when changing the AWS region. The regional endpoint is [recommended](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) because it is faster and has lower latency. In addition, all calls to the regional endpoint are logged in AWS Cloud Trail as any regional service call.
            * Use the [global endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) if the selected regional endpoint is [disabled](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable) on the Amazon account and you do not want to enable it.
            * [Contact the TeamCity support team ](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?) if you need to specify a custom endpoint for Amazon alternatives like [MinIO](https://min.io/).

   IAM Role
   :
   Using [IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), you can delegate access to your AWS resources to users, applications, or services that usually don't have these permissions. These entities will [assume this role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html) to get such access.
   
     You can use the IAM Role as the credentials type only if you already have at least one AWS connection with access keys or default credential provider chain configured in this TeamCity project.
     1. Specify **IAM Role**:
          * **AWS Connection**. Select the AWS connection that will [grant the specified IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html).
          * **Role ARN**. Specify the [ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-arns) of the role to assume by the connection you are creating.
     2. Configure **Session settings**:
          * **Session tag**. The session [tag](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_session-tags.html#id_session-tags_know) is required by Amazon. It is useful to locate sessions created by the TeamCity connection in AWS logs. TeamCity generates the tag automatically, but you can specify your own value.
          * **STS endpoint**.
            * TeamCity generates this field automatically when changing the AWS region. The regional endpoint is [recommended](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) because it is faster and has lower latency. In addition, all calls to the regional endpoint are logged in AWS Cloud Trail as any regional service call.
            * Use the [global endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) if the selected regional endpoint is [disabled](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable) on the Amazon account and you do not want to enable it.
            * [Contact the TeamCity support team ](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?) if you need to specify a custom endpoint for Amazon alternatives like [MinIO](https://min.io/).

     After the connection is created, you can view and copy the automatically generated external connection ID. We strongly recommend that you always add it to the [trust policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html) in AWS to prevent the [confused deputy problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html). This ensures that only authorized TeamCity AWS connections will be able to use the specified IAM Role.

   Default credential provider chain {product="tc"}
   :
   Select this type to provide access credentials according to the [default chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default).
     This approach provides an alternatives to storing credentials in plain text.
     >Note that if your TeamCity server is hosted on an AWS instance that has an associated IAM role granting access to sensitive resources,
      using the **Default Provider Chain** credentials may present a security risk: 
      in this case the TeamCity project administrators who configured this type of connection can access all AWS resources permitted by the role.    
  
     To mitigate this risk, the **Default credential provider chain** credentials type is disabled by default. To enable it, set [the internal property](server-startup-properties.md#TeamCity+Internal+Properties) `teamcity.internal.aws.connection.defaultCredentialsProviderEnabled=true`. (The default value is `false`.)
     No server restart is required after the property is set.
   {product="tc"}
     When this credentials type is used, TeamCity searches for credentials in the following order:
        1. Java System Properties: `aws.accessKeyId` and `aws.secretAccessKey`.
        2. Environment Variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
        3. Web Identity Token credentials from system properties or environment variables.
        4. Credential profiles file in the default location (`~/.aws/credentials`) shared by all AWS SDKs and the AWS CLI. The default location can be overridden via the [`AWS_SHARED_CREDENTIALS_FILE`](aws-credentials.md) environment variable.
        5. Credentials delivered via the Amazon EC2 container service if the `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment variable is set and the security manager has access to it.
        6. Instance profile credentials delivered through the Amazon EC2 metadata service. 
   {product="tc"}

7. Test and save the connection.

Now you can use the credentials provided by this connection in your builds. To do that, configure the [AWS Credentials build feature](aws-credentials.md).

## Amazon ECR

An Amazon ECR (Elastic Container Registry) connection allows accessing private and public AWS registries. With its help, the [Docker Support](docker-support.md) build feature can store Docker images produced by a build to AWS.

Connection settings:

<table>
<tr><td>Setting</td><td>Description</td></tr>

<tr>
<td>

Repository type

</td>
<td>

Choose to connect to a [private](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) or [public](https://docs.aws.amazon.com/AmazonECR/latest/public/what-is-ecr.html) registry.

</td>
</tr>

<tr>
<td>

AWS region

</td>
<td>

(Only for private registries) Select an AWS region where the target resources are located.

</td>
</tr>

<tr>
<td>

Credentials type

</td>
<td>

* __Access key__: select to use preconfigured AWS account access keys. You can find them in the [Identity and Access Management](https://console.aws.amazon.com/iam) section of your AWS console.
* __Temporary credentials__: get [temporary access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) via AWS STS. Such credentials are short-term and can be revoked anytime. They do not belong to a specific user and can be provided on demand — to grant temporary access to specific resources.

</td>
</tr>

<tr>
<td>

IAM role ARN

(_only for Temporary credentials_)

</td>
<td>

Specify a role to be used for generating temporary credentials. You need to [create this role in advance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html) in your AWS console and assign it to all the permissions you need.

</td>
</tr>

<tr>
<td>

External ID

(_only for Temporary credentials_)

</td>
<td>

Specify an [external ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html). We strongly recommend that you always define it when using temporary credentials. This ensures that only TeamCity will be able to use the specified IAM role.

</td>
</tr>

<tr>
<td>

Default credential provider chain

</td>
<td>

Enable this option to automatically find access credentials according to the [default chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default).

This approach is recommended if you do not want to store the credentials anywhere in the TeamCity environment. By default, it will use the values of `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.

</td>
</tr>

<tr>
<td>

Access key ID

</td>
<td>

Specify the access key ID.

See how to get it [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

</td>
</tr>

<tr>
<td>

Secret access key

</td>
<td>

Specify the secret access key.

See how to get it [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

</td>
</tr>

<tr>
<td>

Registry ID

</td>
<td>

Enter your [account ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html#FindingYourAWSId) number.

</td>
</tr>

</table>

## Slack

This type of connection is used to enable notifications via [Slack](https://slack.com/).

Before configuring a Slack connection, you need to create a [Slack app](https://api.slack.com/apps) with the following [bot token scopes](https://api.slack.com/scopes): `channels:read`, `chat:write`, `im:read`, `im:write`, `users:read`, `team:read`, `groups:read`. You can add these in __Features | OAuth & Permissions | Scopes__ of your Slack app.

To ensure your TeamCity server can connect to Slack, specify all the possible endpoint addresses of the server as __Redirect URLs__ in __Features | OAuth \& Permissions__. In most cases, it would be enough to specify the _Server URL_ set in __[Global Settings](configuring-server-url.md)__ in TeamCity. However, if you use a proxy for your TeamCity server but access this server directly, the authentication in Slack might not work unless the server's IP address is also specified in __Redirect URLs__.
{product="tc"}

To ensure your TeamCity server can connect to Slack, specify its address as __Redirect URL__ in __Features | OAuth \& Permissions__.
{product="tcc"}

>See this [Basic app setup](https://api.slack.com/authentication/basics) guide for more details.

Now you can return to TeamCity, add a new Slack connection, and enter the following connection parameters:
* Client ID and secret from the app's __Basic Information__ page
* A [bot user token](https://api.slack.com/docs/token-types#bot) of your app

Save the connection and proceed with adding a [Notifier](notifications.md#Slack+Notifier) build feature.

<anchor name="jetbrains-space-connection"/>

## JetBrains Space
{id="connect-to-jetbrains-space" auxiliary-id="Connect to JetBrains Space"}

>If you are looking for how to integrate your JetBrains Space instance with TeamCity, check out this **[full integration guide](how-to-configure-cicd-for-jetbrains-space.md)**!

This type of connection can be used for:
* Publishing build statuses in [JetBrains Space](https://www.jetbrains.com/space/) with the help of [Commit Status Publisher](commit-status-publisher.md).
* [Authenticating in TeamCity](configuring-authentication-settings.md#JetBrains+Space) with a JetBrains Space account.
* Creating [projects](creating-and-editing-projects.md), [build configurations](creating-and-editing-build-configurations.md), and [VCS roots](configuring-vcs-roots.md) from a JetBrains Space repository.
* Starting builds on [merge requests](pull-requests.md#JetBrains+Space+Merge+Requests) created in a JetBrains Space repository.

Before configuring this connection, you need to create a dedicated application in JetBrains Space:
1. Go to __Administration | Applications__ and click __New application__.
2. Enter a convenient name and save the application.
3. Go to the app's __Authorization__ tab and click __Configure requirements__ under the __In-context Authorization__ section. Enter the name of the Space project you are about to access from TeamCity.
4. Now, you need to set permissions that will be granted to the app in this project. Click __Configure__ and enable the following permissions:
   * Required for authentication and Pull Requests:
      * _Members | View member profile_
   * Required for Commit Status Publisher:
      * _Git Repositories | Report external check status_
   * Required for Pull Requests:
       * _Code Review | View code reviews_
   
   You can approve project-level permissions right in this **Authorization** tab if you are the project's administrator. Global permissions like viewing a member profile require a server administrator's approval.
5. Go back to the app's __Overview__ and open the __Authentication__ tab.
6. Enable _Client Credentials Flow_.
7. To be able to use authentication via Space in TeamCity or/and to create projects/configurations from Space repositories, enable _Authorization Code Flow_ as well. Enter your TeamCity server's URL as the redirect URI.  
   To ensure that your TeamCity server can always connect to JetBrains Space, specify all the other possible endpoint addresses of the server. In most cases, it would be enough to specify the _Server URL_ set in __Global Settings__ in TeamCity. However, if you use a proxy for your TeamCity server but access this server directly, the authentication might not work unless the server's IP address is also specified here.
8. Copy the app's _Client ID_ and _Client secret_.

__Note__: When you create a project in JetBrains Space, it does not automatically add you to this project as a member — this needs to be done manually. TeamCity will be able to see only those projects where you are listed as a member.

Now you can return to TeamCity, add a new JetBrains Space connection, and enter the following connection parameters:
* URL of the Space server
* Client ID and secret of your Space application

Save the connection and proceed with adding a [Commit Status Publisher](commit-status-publisher.md) or [Pull Requests](pull-requests.md#JetBrains+Space+Merge+Requests) feature, [enabling Space authentication](configuring-authentication-settings.md#JetBrains+Space), or creating a [project](creating-and-editing-projects.md#Creating+project+pointing+to+JetBrains+Space) / [build configuration](creating-and-editing-build-configurations.md#Creating+Build+Configuration+Pointing+to+Specific+VCS) / [VCS root](configuring-vcs-roots.md).

## NPM Registry
{id="npm-registry-settings" auxiliary-id="npm-registry-settings"}

This type of connection allows accessing a private [npm registry](https://docs.npmjs.com/cli/v7/using-npm/registry) during a build.

Connection settings:

<table>

<tr><td>Setting</td><td>Description</td></tr>

<tr>

<td>Scope</td>
<td>

Specify an npm user/organization's [scope](https://docs.npmjs.com/cli/v7/using-npm/scope) to [associate](https://docs.npmjs.com/cli/v7/using-npm/scope#associating-a-scope-with-a-registry) with the connected registry. If you want to use multiple registries per project, you need to specify a scope for each of them.

Leave empty if you want to use only one registry in this project. It will be used by `npm`/`yarn` commands by default.

>If you don't specify a scope in the connection settings, you can still access a specific registry within a Node.js step by appending `--registry registry_url` to the `npm ...` command.

</td>

</tr>

<tr>

<td>Registry URL</td>
<td>

Specify the npm registry URL in the following format: `http(s)://hostname[:port]`. For example, `https://npm.pkg.jetbrains.space/mycompany/p/projectkey/mynpm`. The HTTPS schema is used by default.

</td>

</tr>

<tr>

<td>Access token</td>
<td>

Specify a [token](https://docs.npmjs.com/about-access-tokens), if it's needed for accessing the registry. Leave empty for anonymous access. Note that token-based authentication could differ depending on a registry type. See instructions for [npm Enterprise](https://docs.npmjs.com/creating-and-viewing-access-tokens), [Space Packages](https://www.jetbrains.com/help/space/authenticate-in-packages.html#space_account), or [GitHub Packages](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries).

</td>

</tr>

</table>

Save the connection and proceed with adding an [NPM Registry Connection](nodejs.md#Accessing+Private+NPM+Registries) build feature.

## Perforce Administrator Access

This type of connection allows [processing task streams on your Perforce server](perforce-workspace-handling-in-teamcity.md#Cleaning+Workspaces+on+Perforce+Server). In the connection settings, enter the host and user credentials for accessing the Perforce server (the user must have the [admin](https://www.perforce.com/manuals/p4sag/Content/P4SAG/protections.set.html#protections.set.access_levels) permission).

 <seealso>
        <category ref="admin-guide">
            <a href="configuring-vcs-roots.md">Configuring VCS Roots</a>
            <a href="creating-and-editing-projects.md">Creating and Editing Projects</a>
            <a href="creating-and-editing-build-configurations.md">Creating and Editing Build Configurations</a>
        </category>
        <category ref="examples">
            <a href="how-to-configure-cicd-for-jetbrains-space.md">How to Configure CI/CD for JetBrains Space</a>
        </category>
</seealso>