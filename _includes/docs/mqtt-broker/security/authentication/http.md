* TOC
{:toc}

The HTTP Service Authentication Provider allows TBMQ to delegate client authentication to an external REST API.
This is particularly useful for integrating TBMQ with existing identity management systems, custom databases, or legacy authentication services.

### HTTP authentication overview

When an MQTT client attempts to connect, TBMQ suspends the connection and sends an HTTP request to the configured external service.
This request contains the client's credentials (such as username, password, and client ID).

The external service validates these credentials and returns a response indicating whether the connection should be accepted.
Furthermore, the external service can dynamically assign the **Client Type** and **Authorization Rules** in its JSON response.
If the service returns **200 OK** but omits these fields, TBMQ falls back to the default values configured in the provider.

### Configure provider

HTTP authentication is configured through the TBMQ user interface.
This section explains how to configure the endpoint, define the request structure, and set up default authorization rules.

{% include docs/mqtt-broker/user-guide/ui/authentication-provider-control.md %}

#### Endpoint configuration

The core of this provider is the **Endpoint URL** and **Request Method**. TBMQ supports both **GET** and **POST** methods.

* **Endpoint URL**: The full URL of the external authentication service (e.g., `https://api.example.com/mqtt-auth`).
* **Request Method**: The HTTP method used to send the request (GET or POST).

{% include images-gallery.html imageCollection="configure-http-endpoint" %}

#### Request Headers and Body

You can configure custom **Headers** (e.g., `Content-Type`, `Authorization`) to be included in the request.

For the **Request Body**, you can define a JSON object with dynamic placeholders.
How this body is used depends on the **Request Method**:

* **POST**: The JSON object is sent as the request payload.
* **GET**: The keys and values in the JSON object are transformed into URL query parameters (e.g., `?clientId=...&username=...`) appended to the Endpoint URL.

**Supported placeholders:**

* `${clientId}` - The client ID provided by the MQTT client.
* `${username}` - The username provided by the MQTT client.
* `${password}` - The password provided by the MQTT client.

**Example Configuration:**

```json
{
  "u": "${username}",
  "p": "${password}",
  "cid": "${clientId}"
}
```
{: .copy-code}

{% include images-gallery.html imageCollection="configure-http-body" %}

#### Server Response Structure

To fully utilize the HTTP provider, your external server should return a JSON response specifying the authentication result and optional permissions.

**Expected JSON Response Format (case-insensitive):**

```json
{
  "result": "success",
  "clientType": "DEVICE",
  "authRules": {
    "pub": ["telemetry/.*", "alerts/.*"],
    "sub": ["commands/.*"]
  }
}
```
{: .copy-code}

**Fields:**

* **result**:
    * `success` - Client is authenticated.
    * `failure` - Authentication failed (access denied).
    * `skipped` - Authentication skipped; TBMQ will try the next configured provider.
* **clientType**: (Optional) Sets the client type (`APPLICATION` or `DEVICE`). If missing, the **Default Client Type** is used.
* **authRules**: (Optional) A JSON object containing `pub` and `sub` arrays of regex topic patterns. If missing, the **Default Authorization Rules** are used.

#### Authorization Defaults

If your external server returns a simple **200 OK** without a JSON body, or omits specific fields, TBMQ applies the default configurations defined in this section.

* **Default Client Type**: Applied if the server response does not contain the `clientType` field.
* **Default Authorization Rules**: Applied if the server response does not contain the `authRules` field.

{% include images-gallery.html imageCollection="configure-http-authorization" %}

### Advanced settings

The "Advanced settings" section allows you to fine-tune the performance and reliability of the HTTP integration:

* **Read timeout**: The maximum time (in milliseconds) TBMQ waits for a response from the external server before failing authentication.
* **Max parallel requests count**: The maximum number of concurrent authentication requests TBMQ will make to protect the external server.
* **Max response size**: The maximum size (in KB) of the response body accepted from the server.

### Example

In this example, we will use **Beeceptor** to simulate an authentication server that dynamically assigns permissions.

**1. Create a Mock Endpoint**

1.  Go to [Beeceptor](https://beeceptor.com) and create a new endpoint.
2.  Create a **Mocking Rule** for `POST /login`:
    * **Response Status**: 200 OK
    * **Response Body**:

```json
{
  "result": "success",
  "clientType": "DEVICE",
  "authRules": {
    "pub": ["sensors/.*/data"],
    "sub": ["config/global"]
  }
}
```
{: .copy-code}

**2. Configure TBMQ**

1.  Enable the **HTTP Service** provider.
2.  Set **Endpoint URL** to your Beeceptor URL (e.g., `https://your-endpoint.free.beeceptor.com/login`).
3.  Set **Request Method** to `POST`.
4.  Set **Body** to: `{"username": "${username}", "password": "${password}"}`.
5.  Set **Default Client Type** to `APPLICATION` (this will be overridden by the mock response).
6.  Set **Default Auth Rules** to `.*` (this will be overridden by the mock response).

**3. Connect a Client**

Use `mosquitto_pub` to test the connection.

mosquitto_pub -h "YOUR_TBMQ_HOST" -t "sensors/123/data" -m "temp=25" -u "myuser" -P "secret"

*Result:* The client connects. TBMQ receives the JSON response from Beeceptor, assigns the client type **DEVICE**, restricts publishing to `sensors/.*/data`, and allows subscription to `config/global`.

## Next steps

{% assign currentGuide = "SecurityGuide" %}{% include templates/mqtt-broker-guides-banner.md %}