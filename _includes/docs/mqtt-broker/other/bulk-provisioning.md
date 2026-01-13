
* TOC
{:toc}

TBMQ provides a specialized bulk import tool that allows system administrators to manage large numbers of [MQTT client credentials](/docs/{{docsPrefix}}mqtt-broker/user-guide/ui/mqtt-client-credentials/) efficiently using a CSV file.

The bulk import feature implements an **upsert** pattern that automatically identifies existing credentials by their **Name**. If a match is found in the database, the system **updates** the entry with the data from the CSV; otherwise, it **creates** a new one.

Currently, this functionality is specifically designed to support the [**Basic**](/docs/{{docsPrefix}}mqtt-broker/security/#basic-authentication) client credentials type.

## Client Credentials Import

To import multiple credentials, you need to prepare a CSV file where each line corresponds to a single credential record.

The next file was created using a CSV editor and contains data for two credentials.

{% capture tabspec %}client-credentials-file
A,client-credentials-bulk-import.csv,text,resources/client-credentials-bulk-import.csv,/docs/{{docsPrefix}}user-guide/resources/client-credentials-bulk-import.csv{% endcapture %}
{% include tabs.html %}

How the broker processes this file:

**1. Sensor_Device**.
The system creates a new **DEVICE** credentials `Sensor_Device` using the provided Client ID `sensor-01`, Username `mqtt-user-01`, and the description `Demo device client`. It encodes the plain-text Password `secretPass` and applies the topic patterns parsed by the `;` delimiter for both subscribe (`sensors/1/data`, `sensors/all/data`) and publish (`sensors/1/cmd`) permissions.

**2. Application_Manager**.
The system creates a new **APPLICATION** credentials `Application_Manager` with Client ID `app-mgr-01`, leaving the Username, Password, and description fields **null** because their cells are empty. The Sub auth rule pattern is set to **null** which forbids subscription to all topics, while the Pub auth rule is set to the `alerts/#` pattern.

### Step 1: Select a file
Upload your prepared CSV file to the system.

![image](/images/mqtt-broker/other/client-credentials-bulk-import-1.png)

**Note:** The CSV file must contain at least two columns:
1. **Name**.
2. **Username** or **Client ID**.

### Step 2: Import configuration
Configure the following parameters to help the system interpret your data:
* **CSV delimiter**: The character used to separate values in the data line (e.g., `,`, `;`, `|`, or `TAB`).
* **Auth rule patterns delimiter**: A specific delimiter provided via the UI used exclusively for splitting **PUB/SUB authorization rule patterns** within a single cell.
  Please note that the **CSV delimiter** cannot be the same as the **Auth rule patterns delimiter**.
* **First line contains column names**: When enabled, the first line of the file is used to automatically map column names in the next step.

![image](/images/mqtt-broker/other/client-credentials-bulk-import-2.png)

### Step 3: Select columns type
At this stage, you map the columns of your CSV file to the specific data types in TBMQ. Supported column types for Basic credentials include:
* **Name**: Required for identification and upsert logic.
* **Client type**: Defines whether the client is a `DEVICE` or an `APPLICATION`. If missing, the `DEVICE` type will be set **by default**.
* **Client ID**: The unique MQTT client identifier.
* **Username**: The MQTT client username.
* **Password**: The MQTT client password (encoded during import).
* **Subscribe auth rule patterns**: List of allowed subscribe topics.
* **Publish auth rule patterns**: List of allowed publish topics.
* **Description**: Optional text describing the credential.

![image](/images/mqtt-broker/other/client-credentials-bulk-import-3.png)

### Step 4: Processing
The system processes the input data line by line according to your mapping.

### Step 5: Results
Upon completion, the system provides a summary of the execution: the number of **created** entities, **updated** entities, and **errors** encountered.

![image](/images/mqtt-broker/other/client-credentials-bulk-import-4.png)

![image](/images/mqtt-broker/other/client-credentials-bulk-import-5.png)

## Data handling rules

* **To update a field**: Provide a new value in the CSV column (please note the specific security logic for [Password](#password-security-policy)).
* **To preserve existing data**: Do not map the specific column during the import process or leave the cell empty, with the exception of Authorization rule patterns, which follow the [Authorization Patterns Policy](#authorization-rule-patterns-policy).

### Password security policy
For security reasons, **existing passwords are protected** during the bulk import, meaning you **cannot erase or change** them via bulk import:
* **Existing Passwords**: If a credential already has a password set, the broker will **not allow changing it**.
* **New Passwords**: You can only set a password via the CSV file if the credential is being **newly created** or if the existing credential currently has **no password set**.

### Authorization rule patterns policy
The system interprets the presence of `Authorization rule pattern` columns differently to allow for intentional permission clearing.

**New Credentials**:
* **Column is present but empty**: The system creates the credential with **null** authorization rules, which **forbids all topics**.
* **Column is not present (not mapped)**: The system creates the credential with default authorization rules (`.*`), which **allows all topics**.

**Existing Credentials**:
* **Column is present but empty**: The system updates the record with **null** authorization rules, revoking previous permissions.
* **Column is not present (not mapped)**: The system **will not update** the current authorization rules, preserving existing permissions.