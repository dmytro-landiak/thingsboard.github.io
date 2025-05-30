External Nodes used are used to interact with external systems.

* TOC
{:toc}

## AWS Lambda Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 3.7.1</em></strong></td>
     </tr>
   </thead>
</table> 

![Node example image](/images/user-guide/rule-engine-2-0/nodes/external-aws-lambda.png)

Node publishes messages to AWS Lambda, a service that lets you run code without provisioning or managing servers. 
It sends messages using a RequestResponse invocation type. The node uses a pre-configured client and specified function to run.

**Configuration**

![Configuration example image](/images/user-guide/rule-engine-2-0/nodes/external-aws-lambda-config.png)

Function configuration

- **Function name**: required parameter to specify which AWS Lambda function should be invoked.
- **Qualifier**: optional parameter to specify a version or alias of the Lambda function. If the qualifier is not specified, the default qualifier **$LATEST** will be used.

> **Note**: **Function name** and **Qualifier** fields support templatization.

AWS Credentials

- **AWS Access Key ID** and **AWS Secret Access Key** are the credentials of an AWS IAM User with programmatic access.
  More information on AWS access keys can be found [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).
- **AWS Region** must correspond to the one in which the Lambda function is created. Current list of AWS Regions can be found [here](https://docs.aws.amazon.com/general/latest/gr/rande.html).

Advanced settings

- **Connection timeout**: amount of time to wait (in seconds) when initially establishing a connection before giving up and timing out.
A value of 0 means infinity, and is not recommended.
- **Request timeout**: amount of time to wait (in seconds) for the request to complete before giving up and timing out.
A value of 0 means infinity, and is not recommended.
- **Tell Failure if AWS Lambda function execution raises exception**: if enabled, forces failure of message processing if AWS Lambda function execution raises exception. 
If disabled, the error information is added to the response payload, and the message will be routed via success chain.

**Output**
- **Success**: If message was processed successfully.
- **Failure**: If an error occurs during message processing or if the AWS Lambda function execution raises an exception, and the option **Tell Failure if AWS Lambda function execution raises exception** is enabled.

**Usage example: monitoring and processing water meter data with AWS Lambda**

Consider the following scenario: we have a water meter IoT device that sends periodic updates about water usage to our system.
We need to process these updates to check for anomalies and perform specific actions based on the water usage patterns.

Solution with AWS Lambda node:
1. **Receive water meter data**: Our rule chain starts with receiving water meter updates. Each update contains the water usage data.
2. **Preprocess the data**: The message passes through a Transformation Node to format the data appropriately for AWS Lambda.
3. **Invoke AWS Lambda function**: The formatted message is then sent to the AWS Lambda Node. This node is configured with the necessary AWS credentials and function details.
4. **Process data with Lambda**: The AWS Lambda function is invoked, processing the water usage data. It checks for anomalies, such as unusually high water usage, and logs the results or triggers further actions.
5. **Handle Lambda response**: Upon successful execution, the Lambda function returns a response. The AWS Lambda Node captures this response, including the requestId in the message metadata.
6. **Route based on Lambda response**: Based on the response from AWS Lambda, the message is routed through either the Success or Failure path. The Success path can continue to further processing, while the Failure path handles any errors or issues encountered.

By utilizing the AWS Lambda Node in this manner, we can efficiently process and respond to real-time water meter data, leveraging AWS Lambda's capabilities for data processing and anomaly detection.

![Rule chain example image](/images/user-guide/rule-engine-2-0/nodes/external-aws-lambda-chain.png)

**Published payload** - node will publish message payload to the AWS Lambda. If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the AWS Lambda.

**Outbound message** from this node will contain response **requestId** in message metadata.
Message payload will contain result of the function execution.

<br>

## AWS SNS Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sns.png)

Node publish messages to AWS SNS (Amazon Simple Notification Service).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sns-config.png)

- **Topic ARN pattern** - can be set direct topic name for message publishing 
or pattern can be used, that will be resolved to the real ARN Topic name using Message metadata. 
- **AWS Access Key ID** and **AWS Secret Access Key** are the credentials of an AWS IAM User with programmatic access. More information on AWS access keys can be found [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). 
- **AWS Region** must correspond to the one in which the SNS Topic(s) are created. Current list of AWS Regions can be found [here](http://docs.aws.amazon.com/general/latest/gr/rande.html).

In the following example, topic name depends on Device Type and there is a Message that contains **deviceType** field in Metadata:
{% highlight javascript %}
{
    deviceType: controller
}
{% endhighlight %}

For publishing message in **controller**'s topic, we will set this pattern in **Topic ARN pattern**:

{% highlight bash %}
arn:aws:sns:us-east-1:123456789012:${deviceType}
{% endhighlight %}

In runtime, pattern will be resolved to <code>arn:aws:sns:us-east-1:123456789012:controller</code>

**Published payload** - Node will publish full Message payload to the SNS.
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the SNS.

**Outbound message** from this node will contain response **messageId** and **requestId**
 in Message metadata. Original Message payload, type and originator will not be changed.

<br>

## AWS SQS Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sqs.png)

Node publish messages to the AWS SQS (Amazon Simple Queue Service).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sqs-config.png)

- **Queue Type** - SQS queue type. Can be *Standard* or *FIFO*.
- **Queue URL Pattern** - Pattern for building Queue URL. For example <code>${deviceType}</code>.
Can be set direct Queue URL for message publishing or pattern can be used, that will be resolved to the real Queue URL using Message metadata.
- **Delay** - delay in seconds, used to delay a specific message.
- **Message attributes** - optional list of message attributes to publish.
- **AWS Access Key ID** and **AWS Secret Access Key** are the credentials of an AWS IAM User with programmatic access. More information on AWS access keys can be found [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). 
- **AWS Region** must correspond to the one in which the SQS Queue(s) are created. Current list of AWS Regions can be found [here](http://docs.aws.amazon.com/general/latest/gr/rande.html).

In the following example, Queue URL depends on Device Type and there is a Message that contains **deviceType** field in Metadata:

{% highlight json %}
{
    deviceType: controller
}
{% endhighlight %}

For publishing message in **controller**'s Queue, we will set this pattern in **Queue URL pattern**:

{% highlight bash %}
https://sqs.us-east-1.amazonaws.com/123456789012/${deviceType}
{% endhighlight %}

In runtime, pattern will be resolved to <code>https://sqs.us-east-1.amazonaws.com/123456789012/controller</code>

**Published body** - Node will publish full Message payload to the SQS. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the SQS.

**Published attributes** - optional list of attributes can be added for publishing message in SQS. It is a collection of <NAME> - <VALUE> pairs.
Both, NAME and VALUE, could be a static values or patterns that will be resolved using Message metadata.

If **FIFO** queue is selected, then Message ID will be used as **deduplication ID** and Message originator as **group ID**.

**Outbound message** from this node will contain response **messageId**, **requestId**, **messageBodyMd5**, **messageAttributesMd5** 
and **sequenceNumber** in Message metadata. Original Message payload, type and originator will not be changed. 

<br>

## Kafka Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-kafka.png)

Kafka Node sends messages to Kafka brokers. Expects messages with any message type. Will send record via Kafka producer to Kafka server.

Configuration:

{% if docsPrefix == null %}
![image](/images/user-guide/rule-engine-2-0/nodes/external-kafka-config-ce.png)
{% endif %}
{% if docsPrefix == "pe/" or docsPrefix == "paas/" or docsPrefix == "paas/eu/" %}
![image](/images/user-guide/rule-engine-2-0/nodes/external-kafka-config-pe.png)
{% endif %}

- **Topic pattern** - can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>
- **Key pattern** - Use <code>${metadataKey}</code> for value from metadata, <code>$[messageKey]</code> for value from message body.
- **Bootstrap servers** - list of kafka brokers separated with comma.
- **Automatically retry times** - number of attempts to resend message if connection fails.
- **Produces batch size** - batch size in bytes for grouping messages with the same partition.
- **Time to buffer locally** - max local buffering window duration in ms.
- **Client buffer max size** - max buffer size in bytes for sending messages.
- **Number of acknowledgments** - number of acknowledgments node requires to received before considering a request complete.
- **Other properties** - any other additional properties could be provided for kafka broker connection.

**Published body** - Node will send full Message payload to the Kafka topic. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the Kafka.

**Outbound message** from this node will contain response **offset**, **partition** and **topic** properties in the Message metadata. 
Original Message payload, type and originator will not be changed.

**Note** - if you want to use [Confluent cloud](https://confluent.cloud) as a kafka broker you should add next properties:

<table>
    <tr>
        <th>Key</th>
        <th>Value</th>
    </tr>
    <tr>
        <td>ssl.endpoint.identification.algorithm</td>
        <td>https</td>
    </tr>
    <tr>
        <td>sasl.mechanism</td>
        <td>PLAIN</td>
    </tr>
    <tr>
        <td>sasl.jaas.config</td>
        <td>org.apache.kafka.common.security.plain.PlainLoginModule required username="CLUSTER_API_KEY" password="CLUSTER_API_SECRET";</td>
    </tr>
    <tr>
        <td>security.protocol</td>
        <td>SASL_SSL</td>
    </tr>
</table>
- **CLUSTER_API_KEY** - your access key from Cluster settings.
- **CLUSTER_API_SECRET** - your access secret from Cluster settings.

<br>

## MQTT Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-mqtt.png)

Publish incoming message payload to the topic of the configured MQTT broker with QoS **AT_LEAST_ONCE**. 

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-mqtt-config.png)

- **Topic pattern** - can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>.
- **Host** - MQTT broker host.
- **Port** - MQTT broker port.
- **Connection timeout** - timeout in seconds for connecting to MQTT broker.
- **Client ID** - optional client identifier used for connecting to MQTT broker. If not specified, default generated clientId will be used.
- **Add Service ID as suffix to Client ID** - optional flag. Server id will be added as a suffix to the client id when enabled. It is helpful when running in microservices mode to allow rule nodes on each node to connect to the broker without the errors.
- **Clean session** - establishes a non persistent connection with the broker when enabled.
- **SSL Enable/Disable** - enable/disable secure communication.  
- **Credentials** - MQTT connection credentials. Can be either *Anonymous*, *Basic* or *PEM*.

Different Authentication credentials are supported for external MQTT broker:

- Anonymous - no authentication
- Basic - username\password pair is used for authenticating
- PEM - PEM certificates are used for Authentication

If **PEM** credentials type is selected, the following configuration should be provided:

- CA certificate file
- Certificate file
- Private key file
- Private key password

<br>

**Published body** - Node will send full Message payload to the MQTT topic.
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the MQTT broker.

In case of successful message publishing, original Message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

**MQTT retransmission mechanism**

The MQTT node uses ThingsBoard's internal MQTT client.

{% if docsPrefix contains "paas" %}
{% include docs/user-guide/mqtt-retransmission-mechanism.md show-yml-config=false %}
{% else %}
{% include docs/user-guide/mqtt-retransmission-mechanism.md show-yml-config=true %}
{% endif %}

When the message is dropped, the corresponding rule engine message is routed via **Failure** chain with the appropriate exception message.

<br>

## Azure IoT Hub Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.5.3</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-iot-hub.png)

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-iot-hub-config.png)

- **Topic** - for more information about IoT Hub topic use [link](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support#sending-device-to-cloud-messages).
- **Hostname** - Azure IoT Hub Hostname.
- **Device ID** - Your Device ID from Azure IoT Hub.
- **Credentials** - Azure IoT Hub connection credentials. Can be either *Shared Access Signature* or *PEM*.

Different Authentication credentials are supported for Azure IoT Hub:

- Shared Access Signature - SAS Key is used for Authentication
- PEM - PEM certificates are used for Authentication

If **Shared Access Signature** credentials type is selected, the following configuration should be provided:
- SAS Key - it is key from your device in [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-authenticate-downstream-device#symmetric-key-authentication)
- CA certificate file, by default used Baltimore certificate. More about certificates [here](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support#tlsssl-configuration)

If **PEM** credentials type is selected, the following configuration should be provided:

- CA certificate file, by default used Baltimore certificate. More about certificates [here](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support#tlsssl-configuration)
- Certificate file
- Private key file
- Private key password

[X.509 CA-signed authentication](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-authenticate-downstream-device#x509-ca-signed-authentication)

[CACertificates instruction](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates)

<br>

**Published body** - Node will send full Message payload to the Azure IoT Hub device.
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the Azure IoT Hub.

In case of successful message publishing, original Message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

<br>

## RabbitMQ Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-rabbitmq.png)

Publish incoming message payload to the RabbitMQ.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-rabbitmq-config.png)

- **Exchange name pattern** - the exchange to publish the message to. Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code> .
- **Routing key pattern** - the routing key. Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code> .
- **Message properties** - optional routing headers. Supported headers *BASIC*, *TEXT_PLAIN*, *MINIMAL_BASIC*, *MINIMAL_PERSISTENT_BASIC*, *PERSISTENT_BASIC*, *PERSISTENT_TEXT_PLAIN*
- **Host** - default host to use for connections
- **Port** - default port to use for connections
- **Virtual host** - the virtual host to use when connecting to the broker
- **Username** - AMQP user name to use when connecting to the broker
- **Password** - AMQP password to use when connecting to the broker
- **Automatic recovery** - enables or disables automatic connection recovery
- **Connection timeout** - connection TCP establishment timeout in milliseconds; zero for infinite
- **Handshake timeout** - the AMQP0-9-1 protocol handshake timeout, in milliseconds
- **Client properties** - additional properties that are sent to the server during connection startup 

**Published body** - Node will send full Message payload to the RabbitMQ. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload.

In case of successful message publishing, original Message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

<br>

## REST API Call Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-rest-api-call.png)

Invoke REST API calls to the external REST server.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-rest-api-call-config.png)

- **Endpoint URL pattern** - Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>
- **Request method** - *GET*, *POST*, *PUT*, *DELETE*
- **Headers** - request headers, header or value can be a static string, or pattern that is resolved using Message Metadata properties.

**Endpoint URL**

URL can be a static string or a pattern. Only Message metadata is used for resolving patterns. 
So property names that are used in the patterns must exist in the Message Metadata, otherwise raw pattern will be added into URL.

For example, if Message payload contains property **deviceType** with value **container**, then this pattern: 

<code>http://localhost/api/${deviceType}/update</code> 

will be resolved to 

<code>http://localhost/api/container/update</code>   

**Headers**

Collection of header name/value can be configured. Those headers will be added into Rest request. Pattern should be used for configured both header name and header value.
For example <code>${deviceType}</code>. Only Message metadata is used for resolving patterns. 
So property names that are used in the pattern must exist in the Message Metadata, otherwise raw pattern will be added into header. 

**Request body** - Node will send full Message payload to the configured REST endpoint. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload.

**Outbound message** from this node will contain response **status**, **statusCode**, **statusReason** and responce **headers** in the Message metadata.
Outbound Message payload will be the same as response body. Original Message type and originator will not be changed.

<br>

To send a single file as request body, add a field **attachments** to the message **metadata** with file uuid stored in DataBase. In that case any message data will be ignored and only file content will be sent. 
To define a request content type use the header setting like

<code>Content-Type: application/json; charset=UTF-8</code>

Here an example of message **metadata** to send a single file:

{% highlight javascript %}
{
  "attachments": "e18b6950-dfca-11eb-affb-8db134b46d68"
}
{% endhighlight %}

**NOTE**: This is part of [File Storage](/docs/{{docsPrefix}}user-guide/file-storage/) feature supported by [ThingsBoard Professional Edition](/products/thingsboard-pe/).

<br>

In case of successful request, outbound message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

<br>

## Send Email Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-email.png)

Node sends incoming message using configured Mail Server. This Node works only with messages that where created using 
[**To Email**](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/transformation-nodes/#to-email-node) transformation Node, please connect this Node with **To Email** Node using **Success** chain.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-email-config.png)

- **Use system SMTP settings** - if enabled default Mail Server configured on System level will be used
- **Protocol** - Mail Server transport protocol: *SMTP* or *SMTPS*
- **SMTP host** - Mail Server host
- **SMTP port** - Mail Server port
- **Timeout ms** - read timeout in milliseconds
- **Enable TLS** - if true, enables the use of the STARTTLS command (if supported by the server)
- **Username** - username for the account at the mail host, if any
- **Password** - password for the account at the mail host, if any

This Node can work with default Mail Server configured on System level.
Please find more details about [how to configure default System SMTP Settings.](/docs/{{docsPrefix}}user-guide/ui/mail-settings/)

If specific Mail Server is required for this node - disable **Use system SMTP settings** checkbox and configure Mail Server manually.

<br>

Additionally this node can create email attachments if incoming message has prepared **attachments** metadata field with reference to files stored in DataBase.

Multiple attachments supported. Use a comma separated no whitespace references (uuids) to send many files. Here an example of message **metadata**:

{% highlight javascript %}
{
  "attachments": "e18b6950-dfca-11eb-affb-8db134b46d68,17383b4c-6000-4bb8-be04-b1cb15aa18c5"
}
{% endhighlight %}

**NOTE**: This is part of [File Storage](/docs/{{docsPrefix}}user-guide/file-storage/) feature supported by [ThingsBoard Professional Edition](/products/thingsboard-pe/).

<br>

In case of successful mail sending, original Message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

You can see the real life example, where this node is used, in the next tutorial:

- [Send Email](/docs/user-guide/rule-engine-2-0/tutorials/send-email/)

<br>

## Send Notification Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 3.5</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-notification.png)

The node can send notification to recipients group using the template.
The template needs to be of a 'Rule node' type. You can use incoming message data and metadata in your template (see [templatization help page](/docs/{{docsPrefix}}user-guide/notifications/#rule-node) for this notification type).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-notification-config.png)

- **Template** - specify a notification template;
- **Recipients** - notification recipients.

<br>

## Send SMS Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 3.2</em></strong></td>
     </tr>
   </thead>
</table> 

Node is able to construct SMS message based on the metadata fields from the incoming message and send it using AWS SNS or Twilio SMS providers.
We recommend enabling debug mode for the rule node.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-sms.png)

- **Use system SMS provider settings** - if enabled default SMS Provider Server configured on System level will be used.
{% unless docsPrefix contains "paas/" %}
See [SMS Provider](/docs/{{docsPrefix}}user-guide/ui/sms-provider-settings) settings for more details;
{% endunless %}
- **Phone Numbers To template** - Allows to configure multiple phone numbers where the SMS will be sent to. Optionally, you may reference fields from the message metadata.  
- **SMS message template** - Allows to configure body of the SMS message. Optionally, you may reference fields from the message metadata.

This Node can work with default SMS provider configured on System level. 
If SMS message will be sent to all recipients successfully, original Message will be passed to the next nodes via Success chain, otherwise Failure chain is used.

## Twilio SMS Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 2.0.2</em></strong></td>
     </tr>
   </thead>
</table> 

{% assign rulenode = "Twilio SMS" %}{% include templates/pe-rule-node-banner.md %}

![image](/images/user-guide/rule-engine-2-0/nodes/external-twilio-sms.png)

Sends incoming message payload as SMS message via Twilio service.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-twilio-sms-config.png)

- **Phone Number From** - can be set direct phone number as Number From of SMS
                          or pattern can be used, that will be resolved to the real Number From using Message metadata.
- **Phone Numbers To** - comma separated recipient Phone Numbers list. Can be set direct phone numbers or pattern can be used, that will be resolved to the real phone numbers using Message metadata.
- **Twilio Account SID** - your Account Sid at twilio.com/console
- **Twilio Account Token** - your Account Token at twilio.com/console

SMS message will be sent to all recipients taken from **Phone Numbers To** list.

If SMS message will be sent to all recipients successfully, original Message will be passed to the next nodes via **Success** chain, otherwise **Failure** chain is used.

<br>

## Send to Slack Node

<table  style="width:250px;">
   <thead>
     <tr>
	 <td style="text-align: center"><strong><em>Since TB Version 3.5</em></strong></td>
     </tr>
   </thead>
</table> 

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-to-slack.png)

The node create a message based on the incoming data and metadata, and send it via [Slack](https://slack.com/) to a public channel, private channel or direct message.
If chosen to use system Slack settings, the node will take the Slack API token from system settings. Otherwise, you need to specify the token in the node configuration.
Learn more about how to configure Slack settings in Thingsboard [here](/docs/{{docsPrefix}}user-guide/ui/slack-settings/).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-send-to-slack-config.png)

- **Message template** - the template for a Slack message; you may optionally reference fields from incoming message data and metadata;
- **Use system slack settings** - if enabled, the node will take the Slack API token from system settings;
- **Slack API token** - token to integrate Thingsboard with Slack;
- **Slack channel type** - send message via Slack to a public channel, private channel or direct message;
- **Conversation** - public channel, private channel or user in Slack to send message to.
