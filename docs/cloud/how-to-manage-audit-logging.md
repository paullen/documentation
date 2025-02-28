---
id: how-to-manage-audit-logging
title: How to manage Audit Logging in Temporal Cloud
sidebar_label: Audit Logging
description: Create, consume, and manage Audit Logs.
toc_max_heading_level: 4
tags:
- term
- explanation
- guide-context
---

<!-- THIS FILE IS GENERATED. DO NOT EDIT THIS FILE DIRECTLY -->

Audit Logging is a feature of <a class="tdlp" href="/cloud/index#">Temporal Cloud<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is Temporal Cloud?</span><br /><br /><span class="tdlppd">Temporal Cloud is a managed, hosted Temporal environment that provides a platform for Temporal Applications.</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/cloud/index#">Learn more</a></span></span></a> that provides forensic access information at the account level, the user level, and the <a class="tdlp" href="/namespaces#">Namespace<span class="tdlpiw"><img src="/img/link-preview-icon.svg" alt="Link preview icon" /></span><span class="tdlpc"><span class="tdlppt">What is a Namespace?</span><br /><br /><span class="tdlppd">A Namespace is a unit of isolation within the Temporal Platform</span><span class="tdlplm"><br /><br /><a class="tdlplma" href="/namespaces#">Learn more</a></span></span></a> level.

Audit Logging answers "who, when, and what" questions about Temporal Cloud resources.
These answers can help you evaluate the security of your organization, and they can provide information that you need to satisfy audit and compliance requirements.

## Supported integrations

Audit Logging supports the [Amazon Kinesis](https://docs.aws.amazon.com/kinesis/) streaming-data platform.
By using [Amazon Kinesis Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html), you can route Temporal Audit Logs in [Amazon Simple Storage Service](https://docs.aws.amazon.com/s3/) (S3).
We plan to release additional integrations.

## Supported events

The first release of Audit Logging supports Admin Operation events.

### Admin Operation events

The following list specifies both the supported events and the Temporal APIs that emit the logs.

- Account
  - Configure Audit Logging: `UpdateAccount`
  - Configure observability: `UpdateAccount`
- User
  - Create user invitations: `InviteUsers`
  - Delete users: `DeleteUser`
  - Update user account-level Roles: `UpdateUser`
  - Update user Namespace permissions: `UpdateUserNamespacePermissions`
  - Log in user: `UserLogin`
- Namespace
  - Create Namespace: `CreateNamespace`
  - Update Namespace: `UpdateNamespace`
  - Delete Namespace: `DeleteNamespace`
  - Add or update certificates or certificate filters: `UpdateNamespace`
  - Add custom Search Attributes: `UpdateNamespace`
  - Rename custom Search Attribute: `RenameCustomSearchAttribute`
  - Request increase in Retention Period: `UpdateNamespace`

### Audit Log format

The log sent to the Kinesis stream is JSON in the following format:

```json
{
  "emit_time": // Time the operation was recorded
  "level": // Level of the log entry, such as info, warning, or error
  "user_email":  // Email address of the user who initiated the operation
  "operation":  // Operation that was performed
  "details":  // Details of the operation
  "status": // Status, such as OK or error
  "category":  // Category of the log entry: Admin or System
  "version": // Version of the log entry, beginning with 0 and updated when a backfill or resend of the same log occurs
  "log_id": // Unique ID of the log entry
}
```

## Configure Audit Logging

To set up Audit Logging, you must have an Amazon Web Services (AWS) account and set up Kinesis Data Streams.

1. If you don't have an AWS account, follow the instructions from AWS in [Create and activate an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/).
2. To set up Kinesis Data Streams, open the [AWS Management Console](https://aws.amazon.com/console/), search for Kinesis, and start the setup process.

Be aware that Kinesis has a rate limit of 1,000 messages per second and quotas for both the number of records written and the size of the records.
For more information, see [Why is my Kinesis data stream throttling?](https://aws.amazon.com/premiumsupport/knowledge-center/kinesis-data-stream-throttling/)

### Create an Audit Log sink

1. In the Temporal Cloud UI, select **Settings**.
1. On the **Settings** page, select **Integrations**.
1. In the **Audit Logging** card, select **Configure Audit Logs**.
1. On the **Audit Logging** page, choose your **Access method** (either **Auto** or **Manual**).
   - **Auto:** Configure the AWS CloudFormation stack in your AWS account from the Cloud UI.
   - **Manual:** Use a generated AWS CloudFormation template to set up Kinesis manually.
1. In **Kinesis ARN**, paste the Kinesis ARN from your AWS account.
1. In **Role name**, provide a name for a new IAM Role.
1. In **Select an AWS region**, select the appropriate region for your Kinesis stream.

If you chose the **Auto** access method, continue with the following steps:

1. Select **Save and launch stack**.
1. In **Stack name** in the AWS CloudFormation console, specify a name for the stack.
1. In the lower-right corner of the page, select **Create stack**.

If you chose the **Manual** access method, continue with the following steps:

1. Select **Save and download template**.
1. Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/).
1. Select **Create Stack**.
1. On the **Create stack** page, select **Template is ready** and **Update a template file**.
1. Select **Choose file** and specify the template you generated in step 1.
1. Select **Next** on this page and on the next two pages.
1. On the **Review** page, select **Create stack**.

## Consume an Audit Log

After you create an Audit Log sink, wait for the logs to flow into the Kinesis stream.
You should see the first logs 2–10 minutes after you configure the sink.
Subsequent logs arrive every 2 minutes if any actions occurred during that 2-minute window.

:::note

You must configure and implement your own consumer of the Kinesis stream.
For an example, see [Example of consuming an Audit Log](/#example-of-consuming-an-audit-log).

:::

### Example of an Audit Log

The following example shows the contents of an Audit Log.

```json
{"emit_time":"2023-10-24T08:19:41Z","level":"LOG_LEVEL_INFO","user_email":"zhengbo@example.com","operation":"UpdateAccount","details":{"client_ca_fingerprints":["5bb99d14fa602f7d39b7d048674a2251"],"search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
{"emit_time":"2023-10-25T21:16:42Z","level":"LOG_LEVEL_INFO","user_email":"alex@example.com","operation":"DeleteUser","details":{"target_users":["0b741c47-e093-47d1-9b74-f2359129f78f"],"search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
{"emit_time":"2023-11-03T19:31:45Z","level":"LOG_LEVEL_INFO","user_email":"matt@example.com","operation":"InviteUsers","details":{"target_users":["matt@example.net"],"search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
{"emit_time":"2023-11-08T08:06:40Z","level":"LOG_LEVEL_INFO","user_email":"zhengbo@example.com","operation":"UpdateUser","details":{"target_users":["zhengbo@example.net"],"search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
{"emit_time":"2023-11-08T08:14:09Z","level":"LOG_LEVEL_INFO","user_email":"zhengbo@example.com","operation":"UpdateNamespace","details":{"namespace":"audit-log-test.example-dev","client_ca_fingerprints":["f186d0bd971ff7d52dc6cc9d9b6f7644"],"search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
{"emit_time":"2023-11-08T09:20:22Z","level":"LOG_LEVEL_INFO","user_email":"zhengbo@example.com","operation":"UpdateUserNamespacePermissions","details":{"namespace":"audit-log-test.example-dev","search_attribute_update":{},"additional_message":"finished unary call with code OK"},"status":"OK","category":"LOG_CATEGORY_ADMIN"}
**********
```

### Example of consuming an Audit Log

The following Go code is an example of consuming Audit Logs from a Kinesis stream and delivering them to an S3 bucket.

```go
func main() {
   fmt.Println("print audit log from S3")
   cfg, err := config.LoadDefaultConfig(context.TODO(),
      config.WithSharedConfigProfile("your_profile"),
   )
   if err != nil {
      fmt.Println(err)
   }
   s3Client := s3.NewFromConfig(cfg)
   response, err := s3Client.GetObject(
      context.Background(),
      &s3.GetObjectInput{
         Bucket: aws.String("your_bucket_name"),
         Key:    aws.String("your_s3_file_path")})
   if err != nil {
      fmt.Println(err)
   }
   defer response.Body.Close()

   content, err := io.ReadAll(response.Body)

   fmt.Println(string(content))
}
```

The preceding code also prints the logs in the terminal.
The following is a sample result.

```json
{
  "emit_time": "2023-11-14T07:56:55Z",
  "level": "LOG_LEVEL_INFO",
  "user_email": "zhengbo@example.com",
  "operation": "DeleteUser",
  "details": {
    "target_users": ["d7dca96f-adcc-417d-aafc-e8f5d2ba9fe1"],
    "search_attribute_update": {},
    "additional_message": "finished unary call with code OK"
  },
  "status": "OK",
  "category": "LOG_CATEGORY_ADMIN"
}
```

## Troubleshoot Audit Logging

The Audit Logging page of the Temporal Cloud UI provides the current status of an Audit Log sink.

- If an error is detected, a summary of the error appears below the page title.
- If the Audit Log sink is functioning normally, an **On** badge appears next to the page heading.

After an Admin Operation is performed, users can see Audit Log messages flow through Kinesis.

Temporal retains Audit Log information for up to 30 days.
If you experience an issue with an Audit Log sink, we can provide the missing audit information.
Open a support ticket to request assistance.

## Delete an Audit Log sink

When you no longer need Audit Logging, you can delete the Audit Log sink.

1. In the Temporal Cloud UI, select **Settings**.
1. On the **Settings** page, select **Integrations**.
1. In the **Audit Logging** card, select **Configure Audit Logs**.
1. At the bottom of the **Audit Logging** page, choose **Delete**.

After you confirm the deletion, the Audit Log Sink is removed from your account and logs stop flowing to your Kinesis stream.
