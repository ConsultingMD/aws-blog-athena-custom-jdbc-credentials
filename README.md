# Grand Rounds Fork

We've forked this from the upstream `aws-samples` organization to support Athena in our organization. The
[original README content](#connecting-to-amazon-athena-with-federated-identities-using-temporary-credentials)
follows the Grand Rounds-specific content.

### Building

`mvn package` will build the jar and put it in the `target` subdirectory.  I built with JDK 1.10 on Linux (Ubuntu 18.04)

### Releasing

To release, we're doing the bare minimum at the moment:

* Build on your workstation
* Create a `Github Release` with a label that matches conventions already on the repo
* Upload the `.jar` as a build artifact

We're doing it this way rather than integrating to Circle-CI because we don't
think this code will need substantial maintenance.

### Installing

[This Grand Rounds Wiki Page](https://grandrounds.atlassian.net/wiki/spaces/EDS/pages/702218241/How+to+set+up+DataGrip+to+talk+to+our+analytics-production+Athena) is often more up to date than the README you're reading, and may have tricks & tips from others who've done it.

Download the `.jar` for this repo into your `.datagrip` directory (on Linux, this is `~/.datagrip`)

While you're at it, download [AthenaJDBC42-2.0.7.jar](https://s3.amazonaws.com/athena-downloads/drivers/JDBC/SimbaAthenaJDBC_2.0.7/AthenaJDBC42_2.0.7.jar) to the same directory.

### Setting up your tool (in general)

We don't actually use either of the custom credential providers provided by
this jar.  We use the jar this repo constructs to get access to a credential
provider that's not packaged in the Athena 2.0.7 driver, but *is* packaged in
the AWS Java SDK used here.

To use this, you need to add this `jar` along with the Athena `jar` to your tool.

To *test what you're doing*, start by making sure you have a session that can
talk to `Athena`, and that your SQL tool's process sees the environment
variables that describe that session.  So, for example:

```
$ aws-environment analytics-production
Operations Account MFA Token:
$ datagrip &
```

If you don't know about `aws-environment` or have an MFA device set up, you'll
need to work with IT and Platform to do so.

### Setting up DataGrip

Once you've downloaded the jars and done the `aws-environment` thing (and
started datagrip from the window where you did that), you can configure the
driver:

1. Create a "custom driver"
   1. Go File`->`DataSources
   1. Click the + on the top-left and choose "Driver"
   1. Click on the + on the top-*right* of the list of jars ("Driver Files") and add both this repo's jar and the JDBC Driver's jar.
   1. In the `Class` field, select `com.simba.athena.jdbc42.Driver`
1. Then use that driver to construct your database connection. See the Confluence page for up-to-date details, but here's the gist:
   1. On the "general" tab:
      * Do not provide a user name or password
      * Use Class `com.simba.athena.jdbc42.Driver`
   1. On the "advanced" tab, you'll need the following. Note that to configure LogLevel and the S3OutputEncKMSKey,
      you'll need to add `<user defined>` names:

      | **Name** | **Value** |
      |----------|-----------|
      | **AwsCredentialsProviderClass** | **com.amazonaws.auth.EnvironmentVariableCredentialsProvider** |
      | **AwsRegion** | **us-east-1** |
      | **S3OutputEncOption** | **SSE_KMS** |
      | **S3OutputLocation** | **s3://aws-athena-query-results-294397492613-us-east-1** |
      | **UseAwsLogger** | **1** |
      | **LogLevel** | **6** |
      | **S3OutputEncKMSKey** | **273e5666-6ab7-4229-a39c-f0e06a7a563a** |

    1. Once you've done that, go ahead back to the "General" tab and hit the "Test Connection" button; you should reach success.

1. If, when working with the DataGrip console, you get prompted for a user name and password, *do not enter them*. This is a symptom of your session (which was established by `aws-environment`) having expired. Quit out of DataGrip and start over *with the `aws-environment` command*.

### Using the new setup

Once you're set up, whenever you want to run DataGrip, you'll need to open a
Terminal and do:

```
$ aws-environment analytics-production
Operations Account MFA Token:
$ datagrip &
```

The session established by `aws-environment` should last 8 hours.  When it
expires, DataGrip will take a long time on a query, then prompt you for user
name and password.  Don't enter anything; quit and start over.

----

The rest of this document is from the original `aws-samples` README:

# Connecting to Amazon Athena with Federated Identities using Temporary Credentials

Using temporary security credentials ensures that access keys to protected resources in production are not directly hard-coded in the applications. Instead, you rely on AWS Secure Token Service (AWS STS) to generate temporary credentials.
Temporary security credentials work similar to the long-term access key credentials that your Amazon IAM users can use, with the following differences. These credentials are:
 *  Intended for short-term use only. You can configure these credentials to last for anywhere from a few minutes to several hours. After they expire, AWS no longer recognizes them, or allows any kind of access from API requests made with them.
 *	Not stored with the user, but are generated dynamically and provided to the user when requested. When (or even before) they expire, the user can request new credentials, as long as the user requesting them still has permissions to do so.

We list below some of the typical use cases in which your organization may require federated access to Amazon Athena:
1.	Running Queries in Amazon Athena while Using Federation via SAML with Active Directory (AD). Your group requires to run queries in Amazon Athena while federating into AWS using SAML with permissions stored in AD.
2.	Enabling Cross-Account Access to Amazon Athena for Users in Your Organization. A member of your group with access to AWS Account “A” needs to run Athena queries in Account “B”.
3.	Enabling Access to Amazon Athena for a Data Application. A data application deployed on an Amazon EC2 instance needs to run Amazon Athena queries via JDBC.



### Pre-requisites


 * Java 8 is installed
 * SQL workbench is installed on your laptop or Windows EC2 instance.(http://www.sql-workbench.net/Workbench-Build123.zip)

#### SQL Workbench Extended Properties for SAML generated credentials

Property | Value
---------------------------|--------------------------------------------------------------------------------------
AwsCredentialsProviderClass|com.amazonaws.athena.jdbc.CustomIAMRoleAssumptionSAMLCredentialsProvider
AwsCredentialsProviderArguments|*access_key_id,secret_access_key,session token*
S3OutputLocation|s3://*bucket where athena results are stored*
LogPath|*local path on laptop or pc where logs are stored*
LogLevel|*LogLevel 1 thru 6*

#### SQL Workbench Extended Properties for Cross-Account Role Access

Property | Value
---------------------------|-----------------------------------------------------------------------
AwsCredentialsProviderClass|com.amazonaws.athena.jdbc.CustomIAMRoleAssumptionCredentialsProvider
AwsCredentialsProviderArguments|*access_key_id,secret_access_key,Cross Account Role ARN*
S3OutputLocation|s3://*bucket where athena results are stored*

LogPath|*local path on laptop or pc where logs are stored*
LogLevel|*LogLevel 1 thru 6*

#### SQL Workbench Extended Properties for EC2 Instance role

Property | Value
---------------------------|--------------------------------------------------------------------------------------
AwsCredentialsProviderClass|com.simba.athena.amazonaws.auth.InstanceProfileCredentialsProvider
S3OutputLocation|s3://*bucket where athena results are stored*
LogPath|*local path on laptop or pc where logs are stored*
LogLevel|*LogLevel 1 thru 6*
