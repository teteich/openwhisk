---

copyright:
  years: 2017, 2020
lastupdated: "2020-07-17"

keywords: object storage, bucket, package, functions

subcollection: openwhisk

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:external: target="_blank" .external}
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}
{:deprecated: .deprecated}
{:download: .download}
{:gif: data-image-type='gif'}

# Object Storage
{: #pkg_obstorage}

You can extend the functionality of your {{site.data.keyword.openwhisk}} app by integrating with an {{site.data.keyword.cos_full}} instance.
{: shortdesc}

**Before you begin** 
* To learn about {{site.data.keyword.cos_full_notm}}, see the [Getting started tutorial](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage). 
* For more information about setting up the {{site.data.keyword.cos_full_notm}} instance, see [Provision an instance {{site.data.keyword.cos_full_notm}}](/docs/cloud-object-storage/basics?topic=cloud-object-storage-gs-dev#gs-dev-provision).

## Packages
{: #obstorage_packages}

Review the following table for a list of {{site.data.keyword.openwhisk_short}} packages that you can use to work with your {{site.data.keyword.cos_full_notm}} entities.
{: shortdesc}

| Package | Availability | Description |
| --- | --- | --- |
| [{{site.data.keyword.cos_full_notm}} trigger](#pkg_obstorage_ev) | Pre-installed | The {{site.data.keyword.cos_full_notm}} trigger [listens for changes](#pkg_obstorage_ev) to an {{site.data.keyword.cos_full_notm}} bucket. |
| [{{site.data.keyword.cos_full_notm}} package](#pkg_obstorage_install)| Installable | You can use the installable `cloud-object-storage` package to [read, write, and delete](#pkg_obstorage_install) from an {{site.data.keyword.cos_full_notm}} bucket. |

## Setting up the {{site.data.keyword.cos_full_notm}} trigger
{: #pkg_obstorage_ev}

**What is the {{site.data.keyword.cos_full_notm}} trigger?**

The {{site.data.keyword.cos_full_notm}} package is a pre-installed {{site.data.keyword.openwhisk_short}} package that you can use to create triggers to listen for changes to objects in a bucket. When a bucket change occurs, the trigger is fired. You can then create actions and rules to process object changes from the bucket. The trigger is available in the `us-east`, `us-south`, `eu-gb`, `eu-de`, and `jp-tok` regions.

**How does the trigger work?**

Once configured, the trigger listens for changes to a bucket and fires on successful change events. When you create the trigger, you can specify a parameter that filters trigger activations based on the bucket change event type, such as `write` events,`delete` events, or `all` events. You can also filter the trigger activations by object `prefix`, `suffix`, or both.

The trigger is fired for each successful bucket change event. Each object change in a batch request is handled individually. For example: A batch request to delete 200 hundred objects would result in 200 individual delete events and 200 trigger fires. For more information, see [Details and limits](#pkg_cos_limits).

**How do I use the trigger?**

After you create a trigger that listens for change events, you can connect it to a {{site.data.keyword.openwhisk_short}} action or sequence of actions to process the object changes. You can use one of the following methods to create actions that are executed when the trigger is fired:
* You can create actions by using the sample code provided in the [COS SDK](/docs/cloud-object-storage/libraries?topic=cloud-object-storage-sdk-gs). The SDK includes code samples for Go, Node.js, Java, and Python.
* You can write your own [actions](/docs/openwhisk?topic=openwhisk-actions) or [web actions](/docs/openwhisk?topic=openwhisk-actions_web) in the language of your choice.
* You can use the sample JavaScript code provided in the [Connecting an action to the trigger](#cos_feed_action_connect) section.
</br>

**Sample use case:**

With the {{site.data.keyword.cos_full_notm}} trigger, you can listen for changes to GPS street data stored in an {{site.data.keyword.cos_full_notm}} bucket. Then, when changes occur, you can trigger the automatic regeneration of a GPS map so that users can have access to the latest street data for their GPS application.

### Prerequisites for working with the {{site.data.keyword.cos_full_notm}} trigger
{: #cos_changes_pre}

**Before you begin**

You must [create an {{site.data.keyword.cos_full_notm}} service instance ](/docs/cloud-object-storage?topic=cloud-object-storage-gs-dev#gs-dev-provision) and [create a regional bucket](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage#gs-create-buckets) in one of the supported regions. Note that your bucket must be in the same region as your {{site.data.keyword.openwhisk_short}} namespace.

In order to use the {{site.data.keyword.cos_full_notm}} trigger, the following conditions must be met.

* Only [IAM-enabled {{site.data.keyword.openwhisk_short}} namespaces](/docs/openwhisk?topic=openwhisk-namespaces) are supported.
* Your {{site.data.keyword.cos_full_notm}} bucket must be a regional bucket and must be in the same region as your {{site.data.keyword.openwhisk_short}} namespace. Cross-region and single-site buckets are not supported.
* You must [assign the Notifications Manager](#pkg_obstorage_auth) role to your {{site.data.keyword.openwhisk_short}} namespace for your {{site.data.keyword.cos_full_notm}}.

### 1. Assigning the Notifications Manager role to your {{site.data.keyword.openwhisk_short}} namespace
{: #pkg_obstorage_auth}

Before you can create a trigger to listen for bucket change events, you must assign the Notifications Manager role to your {{site.data.keyword.openwhisk_short}} namespace. As a Notifications Manager, {{site.data.keyword.openwhisk_short}} can view, modify, and delete notifications for a Cloud Object Storage bucket. You can assign the Notifications Manager role from either the console or the CLI. Once assigned, the Notifications Manager role cannot be removed from your {{site.data.keyword.openwhisk_short}} namespace. Removing the role disables the trigger.
{: shortdesc}

Only account administrators can assign the Notifications Manager role.
{: note}

**What happens when I assign the Notifications Manager role?**

When you assign the Notifications Manager role to your {{site.data.keyword.openwhisk_short}} namespace, you can then create triggers for any regional buckets in your {{site.data.keyword.cos_full_notm}} instance that are in the same region as your {{site.data.keyword.openwhisk_short}} namespace.

You can assign the Notifications Manager role to all instances of {{site.data.keyword.openwhisk_short}} for all instances {{site.data.keyword.cos_full_notm}}, but you can only create triggers for regional buckets that are in the same supported regions as your {{site.data.keyword.openwhisk_short}} namespaces.
{: note}

| Assigning the Notifications Manager role with the console |
|:-----------------|
| <p><ol><li> Navigate to the **Grant a Service Authorization** page in the [IAM dashboard](https://cloud.ibm.com/iam/authorizations/grant){: external}.</li><li> From **Source service**, select **Functions**. Then, from **Source service instance**, select a {{site.data.keyword.openwhisk_short}} namespace. Note: Only IAM-enabled namespaces are supported.</li><li> In **Target service**, select **Cloud Object Storage**, then from **Target service instance**, select your {{site.data.keyword.cos_full_notm}} instance.</li><li> Assign the **Notifications Manager** role and click **Authorize**.</li></ol></p> |
{: caption="Assigning the Notifications Manager role with the console." caption-side="top"}
{: #nm-1}
{: tab-title="Console"}
{: tab-group="notifications"}
{: class="simple-tab-table"}

| Assigning the Notifications Manager role with the CLI |
|:-----------------|
| <p><ol><li>Copy the following command to assign the Notifications Manager role to your {{site.data.keyword.openwhisk_short}} namespace.</li><li> Replace the `source_service_instance_name` variable with the name of your {{site.data.keyword.openwhisk_short}} namespace.</li><li> Replace the `target_service_instance name` variable with the name of your {{site.data.keyword.cos_full_notm}} instance.<pre class="pre"><code>ibmcloud iam authorization-policy-create functions</br> cloud-object-storage "Notifications Manager"</br> --source-service-instance-name &lt;source_service_instance_name&gt;</br> --target-service-instance-name &lt;target_service_instance_name&gt;</code></pre></li><li>Verify the Notifications Manager role has been set.<pre class="pre"><code>ibmcloud iam authorization-policies</code></pre></li></ol></p> |
{: caption="Assigning the Notifications Manager role with the CLI." caption-side="top"}
{: #nm-2}
{: tab-title="CLI"}
{: tab-group="notifications"}
{: class="simple-tab-table"}

</br>

### 2. Determining your trigger parameters
{: #pkg_obstorage_ev_trig_param}

The {{site.data.keyword.cos_full_notm}} trigger includes multiple parameters that can be set to filter which bucket change events fire the trigger. For example, you can configure the trigger to fire on all bucket change events. Or, you can filter trigger fires based on the bucket change event type, such as `write`, `delete`, or `all`  events. You can also filter the trigger activations by object `prefix` or `suffix` or both. You can then create actions and rules to process object changes from the bucket. 
{: shortdesc}

When creating a trigger from the CLI, you can also specify an `endpoint` parameter  When not specified, this parameter is set to the schemeless private regional endpoint of your bucket. Example:
* (Default) Private schemeless endpoint: `s3.private.us-south.cloud-object-storage.appdomain.cloud`. 
* Endpoint with the `https://` scheme: `https://s3.private.us-south.cloud-object-storage.appdomain.cloud`.

For a complete list of available parameters, see [{{site.data.keyword.cos_full_notm}} trigger parameters](#pkg_obstorage_ev_ch_ref_trig) section.
{: note}

### 3. Creating a trigger to listen for bucket changes
{: #pkg_obstorage_ev_trig_ui}

You can create a trigger that responds to {{site.data.keyword.cos_full_notm}} events from the {{site.data.keyword.openwhisk_short}} console or the CLI.
{: shortdesc}

| Creating a trigger with the console |
|:-----------------|
| <p><ol><li> Navigate to the {{site.data.keyword.openwhisk_short}} [**Triggers** page](https://cloud.ibm.com/functions/triggers){: external}.</li><li>Specify an IAM-enabled namespace to contain your entities. Note: Your namespace must be in the same region as your bucket.</li><li>Click **Create**.</li><li> On the **Connect Trigger** page, click **Cloud Object Storage**.</li><li> On the **New Trigger Configuration** page, give your trigger a name.</li><li> From **COS Instance**, select your {{site.data.keyword.cos_full_notm}} instance.</li><li> From **Bucket**, select your {{site.data.keyword.cos_full_notm}} bucket.</li><li> Select the **Object operations** that activates the trigger. Both **Write** and **Delete** are selected by default.</li><li> (Optional) Enter an **Object prefix** or **Object suffix** or both to activate the trigger only when specific objects are updated.</li><li> Click **Create** to create the trigger.</li></ol></p> |
{: caption="Creating a trigger with the console." caption-side="top"}
{: #trigger-1}
{: tab-title="Console"}
{: tab-group="trigger"}
{: class="simple-tab-table"}

| Creating a trigger with the CLI |
|:-----------------|
| <p><ol><li> [Target your IAM-enabled namespace](/docs/openwhisk?topic=openwhisk-namespaces#targeting-namespaces). Note: Your namespace must be in the same region as your bucket.<p><pre class="pre"><code>ibmcloud fn property set --namespace &lt;namespace_name&gt;</pre></code></p></li><li> Create a trigger named `cosTrigger`. The `--param bucket` flag is required. Replace the `<bucket_name>` variable with the name of your bucket. (Optional) You can configure the trigger to fire only on `write` or `delete` events by specifying the `--param event_types` flag. If the `<event_type>` parameter is not specified, the trigger fires on all write, update, and delete changes to your {{site.data.keyword.cos_full_notm}} bucket.<p><pre class="pre"><code>ibmcloud fn trigger create cosTrigger --feed /whisk.system/cos/changes</br> --param bucket &lt;bucket_name&gt;</br> --param event_types &lt;event_type&gt;</br> --param prefix &lt;prefix&gt;</br> --param suffix &lt;suffix&gt;</br> --param endpoint &lt;endpoint&gt;</code></pre></p></li><li> Verify the trigger was created by running the `trigger get` command.<p><pre class="pre"><code>ibmcloud fn trigger get cosTrigger</code></pre></p></li></ol></p> |
{: caption="Creating a trigger with the CLI." caption-side="top"}
{: #trigger-2}
{: tab-title="CLI"}
{: tab-group="trigger"}
{: class="simple-tab-table"}

## Connecting a {{site.data.keyword.openwhisk_short}} action to the trigger
{: #cos_feed_action_connect}

After you create a trigger that listens for change events, you can connect it to a {{site.data.keyword.openwhisk_short}} action or sequence of actions. You can use one of the following methods to create actions:
* You can create actions by using the sample code provided in the [COS SDK](/docs/cloud-object-storage/libraries?topic=cloud-object-storage-sdk-gs). The SDK includes code samples for Go, Node.js, Java, and Python.
* You can write your own [actions](/docs/openwhisk?topic=openwhisk-actions) or [web actions](/docs/openwhisk?topic=openwhisk-actions_web) in the language of your choice.
* You can use the sample JavaScript code provided in the following steps.

### 1. Creating an action to process the trigger results
{: #cos_feed_action}
You can use the console or CLI to create an action that is invoked by the trigger.

**Before you begin**</br>
Copy the following code and save it into a file called `cosChange.js`. This code is used to create an action that is invoked when the trigger is fired. When you make an action using this code, the action returns the data from the bucket change event.

```javascript
function main(data) {
    console.log(data);
}
```
{: codeblock}

| Creating an action with the console |
|:-----------------|
| <p><ol><li> Navigate to the {{site.data.keyword.openwhisk_short}} [**Triggers** page](https://cloud.ibm.com/functions/triggers){: external}.</li><li> Click the trigger that you created.</li><li> On the **Connected Actions** page, click **Add**.</li><li> On the **Add Action** page, select **Create New**.</li><li> Specify a name for your action.</li><li> Select a package for your action.</li><li> For the **Runtime**, select `Node.js 10`.</li><li> Click **Create & Add**.</li><li> On the **Connected Actions** page, click the action that you created.</li><li> On the **Action** page, replace the `Hello World` example code with the code from your `cosChange.js` file.</li><li> Click **Save**.</li></ol></p> |
{: caption="Creating an action with the console." caption-side="top"}
{: #action-1}
{: tab-title="Console"}
{: tab-group="action"}
{: class="simple-tab-table"}

| Creating an action with the CLI |
|:-----------------|
| <p><ol><li>Create an action called `cosChange` by using the `cosChange.js` code.<p><pre class="pre"><code>ibmcloud fn action create cosChange &lt;filepath&gt;/cosChange.js</code></pre></p></li><li> Create a rule called `cosRule` to connect the `cosChange` action to the `cosTrigger` trigger.<p><pre class="pre"><code>ibmcloud fn rule create cosRule cosTrigger cosChange</code></pre></p></li></ol></p> |
{: caption="Creating an action with the CLI." caption-side="top"}
{: #action-2}
{: tab-title="CLI"}
{: tab-group="action"}
{: class="simple-tab-table"}

</br>

### 2. Testing the trigger and action
{: #pkg_obstorage_ev_test}
After you [create a trigger to respond to bucket changes](#pkg_obstorage_ev_trig_ui) and [connect an action to the trigger](#cos_feed_action), you can test that the action is executed as a result of the trigger firing. You can perform this test in either the console or CLI.

| Testing with the console |
|:-----------------|
| <p><ol><li> Make a change to an object in your {{site.data.keyword.cos_full_notm}} bucket. For more information about adding an object to your bucket, see [Add some objects to your bucket](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage#gs-add-objects).</li><li> Navigate to the [{{site.data.keyword.openwhisk_short}} dashboard](https://cloud.ibm.com/functions/dashboard).</li><li> Click **Monitor**.</li><li> Review the **Activity Log** pane for activations of your the trigger and action you created.</li></ol></p> |
{: caption="Creating an action with the console." caption-side="top"}
{: #test-1}
{: tab-title="Console"}
{: tab-group="test"}
{: class="simple-tab-table"}

| Testing with the CLI |
|:-----------------|
| <p><ol><li>Start polling for activations by running the `activation poll` command.<p><pre class="pre"><code>ibmcloud fn activation poll</code></pre></p></li><li> In your {{site.data.keyword.cos_full_notm}} dashboard, either modify an existing bucket object or create one. To learn how to add an object to your bucket, see [Add some objects to your bucket](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage#gs-add-objects).</li><li> For each bucket object change, observe new activations for the `cosTrigger` trigger and `cosChange` action.</li><li> Stop polling by pressing `ctrl + c`.</li><li> You can see the details of an activation by running the `activation get` command.<p><pre class="pre"><code>ibmcloud fn activation get &lt;activation_id&gt;</code></pre></p></li><li>If you are unable to observe new activations, verify that the parameter values are correct by running the `trigger get` command.<p><pre class="pre"><code>ibmcloud fn trigger get cosTrigger</code></pre></p>**Note:** You can see an example activation in the [Data structure of an Object Storage trigger activation](#pkg_obstorage_ev_data) section.</ol></p> |
{: caption="Creating an action with the CLI." caption-side="top"}
{: #test-2}
{: tab-title="CLI"}
{: tab-group="test"}
{: class="simple-tab-table"}

### Next steps
{: #pkg_obstorage_next}

After you have created a trigger to respond to bucket events and connected it to an action, you can try creating custom actions and sequences. 
  * You can use the [COS SDK](/docs/cloud-object-storage/libraries?topic=cloud-object-storage-sdk-gs) to perform bucket and object-level tasks. The SDK includes code samples in multiple languages.
  * You can create your own [actions](/docs/openwhisk?topic=openwhisk-actions) or [web actions](/docs/openwhisk?topic=openwhisk-actions_web) to be executed when the trigger is fired.
  * You can use the actions in the [{{site.data.keyword.cos_full_notm}} package](#pkg_obstorage_install) to [read and write objects to a bucket](#pkg_obstorage_actions) and other tasks. The actions are executed in either Python or Node.js.

### Reference
{: #pkg_obstorage_ev_ch_ref}

Review the following reference material for information on the {{site.data.keyword.cos_full_notm}} trigger package.

#### {{site.data.keyword.cos_full_notm}} trigger entities
{: #pkg_obstorage_ref_trig_entities}

The {{site.data.keyword.cos_full_notm}} trigger contains the `/whisk.system/cos` package and supports the following parameters:

| Entity | Type | Description |
| --- | --- | --- | --- |
| `/whisk.system/cos` | Package | This package contains the `/whisk.system/cos/changes` feed. |
| `/whisk.system/cos/changes` | Feed | This feed responds to changes to an {{site.data.keyword.cos_full_notm}} bucket and fires a {{site.data.keyword.openwhisk_short}} trigger. When you create a trigger using this feed, you can specify [these parameters](#pkg_obstorage_ev_ch_ref_trig) using the `--param` flag. |
{: shortdesc}

#### {{site.data.keyword.cos_full_notm}} trigger parameters
{: #pkg_obstorage_ev_ch_ref_trig}

The `/whisk.system/cos/changes` feed supports the following parameters.

| Parameter | Description |
| --- | --- |
| `bucket` | (Required) The name of of your {{site.data.keyword.cos_full_notm}} bucket. This parameter is required to configure the `changes` feed. The bucket must be in the same region as your {{site.data.keyword.openwhisk_short}} namespace. The bucket must also be configured for regional resiliency. |
| `endpoint` | (Optional). The `endpoint` parameter is the endpoint of your bucket. When not specified, this parameter is set to the schemeless private regional endpoint of your bucket. Example regional `us-south` private schemeless endpoint: `s3.private.us-south.cloud-object-storage.appdomain.cloud`. Example regional `us-south` private endpoint with `https://` scheme: `https://s3.private.us-south.cloud-object-storage.appdomain.cloud`. For more information, see the [Regional endpoints](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints#endpoints-region) table for {{site.data.keyword.cos_full_notm}}.  |
| `prefix` | (Optional). The `prefix` parameter is the prefix of the {{site.data.keyword.cos_full_notm}} objects. You can specify this flag when creating your trigger to filter trigger events by object name prefix. |
| `suffix` | (Optional). The `suffix` parameter is the suffix of your {{site.data.keyword.cos_full_notm}} objects. You can specify this flag when creating your trigger to filter trigger events by object name suffix. |
| `event_types` | (Optional). The `event_types` is the type of bucket change that fires the trigger. You can specify `write` or `delete` or `all`. The default value is `all`. |

#### Data structure of an {{site.data.keyword.cos_full_notm}} trigger activation
{: #pkg_obstorage_ev_data}

The content of the generated events has the following parameters:

| Parameter | Description |
| --- | --- |
| `bucket`| The name of the {{site.data.keyword.cos_full_notm}} bucket that was updated. |
| `object_name` | The name of the object that was changed. |
| `event_type` | The type of event that occurred. |
| `endpoint` | The {{site.data.keyword.cos_full_notm}} endpoint used to connect to the {{site.data.keyword.cos_full_notm}} bucket. This is the endpoint value specified during trigger creation. |
| `key` | The name of the changed object. |

</br>

**Example JSON response of a trigger activation**

You can get details of an activation by running `ibmcloud fn activation get <activation_id>`.

```json
{
    "namespace": "e8b676ee-1726-4541-8f16-f87902bb3ab31",
    "name": "cosTrigger",
    "version": "0.0.1",
    "subject": "ServiceId-65348247-951f-4f22-b2ef-7782597c3ab1",
    "activationId": "78fg525e45964bcdae525e45966bcd6f",
    "start": 1567608038827,
    "end": 0,
    "duration": 0,
    "statusCode": 0,
    "response": {
        "status": "success",
        "statusCode": 0,
        "success": true,
        "result": {
            "bucket": "bucket_name",
            "endpoint": "s3.private.us-east.cloud-object-storage.appdomain.cloud",
            "key": "sample.txt",
            "notification": {
                "bucket_name": "bucket_name",
                "content_type": "application/octet-stream",
                "event_type": "Object:Write",
                "format": "2.0",
                "object_etag": "a2b2d66938b1f023dec6394f12b782b5",
                "object_length": "5",
                "object_name": "sample.txt",
                "request_id": "216c7ddb-218c-4fb7-84d2-293f286b62e6",
                "request_time": "2019-09-04T14:40:35.294Z"
            }
        }
    },
    "logs": [
        "{\"statusCode\":0,\"success\":true,\"activationId\":\"1a80c8449e7d49fb80c8449e7d99fb2a\",\"rule\":\"e8b676ee-1726-4541-8f16-f87902bb3ab31/cosRule\",\"action\":\"e8b676ee-1726-4541-8f16-f87902bb3ab31/cosChange\"}"
    ],
    "annotations": [],
    "publish": false
}
```
{: codeblock}

#### Details and limits
{: #pkg_cos_limits}
Triggers created with `/whisk.system/cos` package have the following limitations.
* The trigger is available in the `us-east`, `us-south`, and `eu-gb` regions.
* Trigger ordering is not guaranteed. Trigger firing sequence may not match bucket update sequence.
* The trigger only fires on successful bucket events.
* Automatic trigger disablement - permissions change, COS bucket authentication changes, namespace authentication.
* Namespaces that contain the `/whisk.system/cos` package cannot be deleted until the package is deleted.
* After the trigger is created, the Notifications Manager role cannot be removed from your {{site.data.keyword.openwhisk_short}} namespace. Removing the role disables the trigger.
* For batch requests, each object change is handled individually and the trigger is fired for each successful change event.
* All characters are permitted in an object key except for ASCII control character NUL. 
* Naming limitations for {{site.data.keyword.openwhisk_short}} triggers can be found on the [System details and limits](/docs/openwhisk?topic=openwhisk-limits#limits_fullnames) page.



</br>

## Configuring the {{site.data.keyword.cos_full_notm}} package
{: #pkg_obstorage_configure}

After you have [created an {{site.data.keyword.cos_full_notm}} service instance ](/docs/cloud-object-storage?topic=cloud-object-storage-gs-dev#gs-dev-provision) and [created at least one bucket](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage#gs-create-buckets), you can install the {{site.data.keyword.cos_full_notm}} package into your namespace to work with your buckets and objects.
{: shortdesc}

The installable {{site.data.keyword.cos_full_notm}} package deploys a set of pre-built actions that you can use to work with your {{site.data.keyword.cos_full_notm}} buckets and objects. These actions are executed in either Node.js or Python. You can select a runtime when you install the package. If you want to use a different runtime, you can use the [COS SDK](/docs/cloud-object-storage/libraries?topic=cloud-object-storage-sdk-gs). You can also [build your own actions](/docs/openwhisk?topic=openwhisk-actions) or [web actions](/docs/openwhisk?topic=openwhisk-actions_web) to respond to the trigger.

For a list of the actions in the `cloud-object-storage` package, see [Available entities](#pkg_obstorage_actions).

If you plan to use the `client-get-signed-url` action, you must [bind your service credentials](#pkg_obstorage_sc_bind) to the package or to the `client-get-signed-url` action. If you do not plan to use this action, you can [set an IAM access policy](#pkg_obstorage_rw_auth) for your {{site.data.keyword.openwhisk_short}} namespace.
{: note}

### 1. Installing the {{site.data.keyword.cos_full_notm}} package
{: #pkg_obstorage_install}

You can install the `cloud-object-storage` package from the console or the CLI.

| Installing with the console |
|:-----------------|
| <p><ol><li> In the {{site.data.keyword.openwhisk_short}} console, go to the [Create page](https://cloud.ibm.com/functions/create){: external}.</li><li> Select the namespace in which you want to install the {{site.data.keyword.cos_full_notm}} package.</li><li> Click **Install Packages**.</li><li> Click the **{{site.data.keyword.cos_full_notm}}** Package group, then click the **{{site.data.keyword.cos_full_notm}}** Package.</li><li> In the **Available Runtimes** section, select either `Node.JS` or `Python`. Then, click **Install**.</li><li> After the package is installed, you are redirected to the **Actions** page and can search for your new package, which is named `cloud-object-storage`.</li></ol></p> |
{: caption="Installing with the console." caption-side="top"}
{: #install-1}
{: tab-title="Console"}
{: tab-group="install"}
{: class="simple-tab-table"}

| Installing with the CLI |
|:-----------------|
| <p><ol><li> Clone the {{site.data.keyword.cos_full_notm}} package repo.<p><pre class="pre"><code> git clone https://github.com/ibm-functions/package-cloud-object-storage.git </code></pre></p></li><li> Navigate to either the `runtimes/nodejs` or `runtimes/python` directory to select a runtime for the actions in the package.<p><pre class="pre"><code>cd package-cloud-object-storage/runtimes/&lt;nodejs_or_python&gt;</code></pre></p></li><li> Deploy the package.<p><pre class="pre"><code>ibmcloud fn deploy</code></pre></p></li><li> Verify that the `cloud-object-storage` package is added to your package list.<p><pre class="pre"><code>ibmcloud fn package list</code></pre></p></li></ol></p> |
{: caption="Installing with the CLI." caption-side="top"}
{: #install-2}
{: tab-title="CLI"}
{: tab-group="install"}
{: class="simple-tab-table"}


### 2. Setting an IAM access policy for your {{site.data.keyword.openwhisk_short}} namespace
{: #pkg_obstorage_rw_auth}

Before you can read or write objects to a bucket, you must provide service-to-service authentication. You can do this by setting an IAM access policy. 

You can create an access policy at either [the service level or the bucket level](/docs/cloud-object-storage/iam?topic=cloud-object-storage-iam-bucket-permissions#iam-service-id). You can do this from either the console or the CLI.

| Setting an IAM access policy with the console |
|:-----------------|
| <p><ol><li> Navigate to the **Service IDs** tab in the IAM console. </li><li> Click the service ID that corresponds to your {{site.data.keyword.openwhisk_short}} namespace.</li><li> On the **Manage** page, click **Access policies**. </li><li> Click **Assign access**. </li><li> Click **Assign access to resources**. </li><li> From **Services**, select **Cloud Object Storage**. </li><li> From **Service instance**, select your {{site.data.keyword.cos_full_notm}} instance. </li><li> To limit access to a specific bucket, enter **bucket** for **Resource type** and enter your **bucket-name** for **Resource ID**.</li><li> Review the [IAM roles](/docs/cloud-object-storage/iam?topic=cloud-object-storage-iam-bucket-permissions#iam-service-id) and select the appropriate role to assign to your namespace.</li><li>After you have selected a role, click **Save**.</li> </ol></p> |
{: caption="Setting an IAM access policy with the console." caption-side="top"}
{: #iam-1}
{: tab-title="Console"}
{: tab-group="iam"}
{: class="simple-tab-table"}

| Setting an IAM access policy with the CLI |
|:-----------------|
| <p><ol><li> Copy the following command and replace the variables.<p><pre class="pre"><code> ibmcloud iam service-policy-create &lt;service-id-name&gt;</br> --roles &lt;role&gt;</br> --service-name cloud-object-storage</br> --service-instance &lt;resource-instance-id&gt;</br> --region global</br> --resource-type bucket</br> --resource &lt;bucket-name&gt; </code></pre></p></li><li> Replace `service-id-name` with the name of your {{site.data.keyword.openwhisk_short}} namespace.</li><li> Replace `role` with the [IAM role](/docs/account?topic=account-userroles) you want to assign to your {{site.data.keyword.openwhisk_short}} namespace.</li><li> (Optional) Include the `--resource-type bucket` and `--resource <bucket-name>` flags to limit access to a specific bucket. If you do not include these flags, access is granted at the service level.</li><li>Replace `<bucket-name>.` with the name of your bucket.</li></ol></p> |
{: caption="Setting an IAM access policy with the CLI." caption-side="top"}
{: #iam-2}
{: tab-title="CLI"}
{: tab-group="iam"}
{: class="simple-tab-table"}

You still need to pass the bucket name and endpoint during action invocation. You can avoid having to pass these values each time by binding them to the `cloud-object-storage` package or to a specific action in the package. To do this, see [Setting default parameters for a package or an action](#pkg_obstorage_param_bind).
{: note}

#### Creating service credentials for accessing your {{site.data.keyword.cos_full_notm}} instance
{: #pkg_obstorage_sc_cos}

If you plan to use the `client-get-signed-url` action, you must create [service credentials](/docs/cloud-object-storage/iam?topic=cloud-object-storage-service-credentials). You can then [bind service credentials](#pkg_obstorage_sc_bind) to the `cloud-object-storage`package.

#### Binding your {{site.data.keyword.cos_full_notm}} service credentials to the package or actions.
{: #pkg_obstorage_sc_bind}

To use the `client-get-signed-url` action in the `cloud-object-storage` package, you must bind your {{site.data.keyword.cos_full_notm}} service credentials to the action.

  * To bind service credentials to all actions in the package, use the `service bind` command in the CLI. 
  * To bind service credentials to individual actions, complete the following steps in the console. 

| Binding service credentials in the console |
|:-----------------|
| <p><ol><li> From the [**Actions** page](https://cloud.ibm.com/functions/actions){: external}, click `cloud-object-storage` package.</li><li>Next, click the `client-get-signed-url` action, then click **Parameters**.</li><li> Click **Add** to enter a new parameter. Note that parameters must be entered as `key` and `value` pairs. <li>For **Parameter Name**, enter the key `__bx_creds`.</li><li> For **Default Value**, paste in the service credentials JSON object from the {{site.data.keyword.cos_full_notm}} service instance that you created earlier.</li><li> Repeat the steps for each action that you want to use.</li></ol></p> |
{: caption="Binding service credentials in the console." caption-side="top"}
{: #service-1}
{: tab-title="Console"}
{: tab-group="service"}
{: class="simple-tab-table"}

| Binding service credentials in the CLI |
|:-----------------|
| <p><ol><li> Bind the credentials from the {{site.data.keyword.cos_full_notm}} instance you created to the package. You can include the `--keyname` flag to bind specific service credentials. For more information about binding services, see [Service commands](/docs/openwhisk?topic=cloud-functions-cli-plugin-functions-cli#cli_service).<p><pre class="pre"><code>ibmcloud fn service bind cloud-object-storage cloud-object-storage --keyname &lt;service_key&gt;</code></pre></p></li><li> Verify that the package is configured with your {{site.data.keyword.cos_full_notm}} service instance credentials.<p><pre class="pre"><code>ibmcloud fn package get &lt;namespace&gt;cloud-object-storage parameters</code></pre></p></li></ol></p> |
{: caption="Binding service credentials in the CLI." caption-side="top"}
{: #service-2}
{: tab-title="CLI"}
{: tab-group="service"}
{: class="simple-tab-table"}

You still need to pass the `bucket` and `endpoint` values during action invocation. Or, you can bind these parameters to the package or an action. To do this, see [Setting default parameters for a package or action](#pkg_obstorage_param_bind).
{: note}

### Setting default parameters for a package or action
{: #pkg_obstorage_param_bind}

Rather than manually passing your `bucket` and `endpoint` with each action invocation, you can use the [`package update`](/docs/openwhisk?topic=cloud-functions-cli-plugin-functions-cli#cli_pkg_bind) command to bind your bucket and endpoint parameters to a specific action or to the `cloud-object-storage` package. You can find your `<bucket_endpoint>` value on the **Endpoint** tab in the COS console. 

For a list of endpoints, see [Endpoints and storage locations](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints#endpoints).

To set a default `bucket-name` and bucket-endpoint value, copy one of the following commands. Replace `<bucket-name>` with the name of your bucket and replace `<bucket_endpoint>` with the endpoint of your bucket. 

When you update parameters for a package, action, or trigger you must specify all previously created parameters. Otherwise, the previously created parameters are removed. Any services that were bound to the package are also removed, so after you update other parameters you must [bind services](/docs/openwhisk?topic=openwhisk-services) to your package again.
{: important}

**Updating parameters for all actions in a package**
```
ibmcloud fn package update cloud-object-storage 
--param bucket <bucket-name> 
--param endpoint <bucket-endpoint>
```
{: pre}

**Updating parameters to a specific action**
```
ibmcloud fn action update cloud-object-storage/object-write 
--param bucket <bucket-name> 
--param endpoint <bucket_endpoint>
```
{: pre}

You can also bind parameters to actions by using **Parameters** in the console. To add parameters in the console, navigate to the [**Actions** page](https://cloud.ibm.com/functions/actions){: external} and click one of your actions. Then, click **Parameters** > **Add Parameter**. You must add parameters in `<key>` and `<value>` pairs.
{: tip}

After you have updated the package or action to include the `bucket` and `endpoint` parameters, you must [rebind the service](#pkg_obstorage_sc_bind) with `service bind` command.

</br>

## Writing an object to a bucket
{: #pkg_obstorage_write}

You can use the `object-write` action to write an object to an {{site.data.keyword.cos_full_notm}} bucket. You can use this action from the console or the CLI. 
{: shortdesc}

In order to write an object, you must specify the `bucket-name`, `object-name`, `body`, and `endpoint`.

If you [bound your `bucket` and `endpoint` parameters](#pkg_obstorage_param_bind) to the `cloud-object-storage` package or to the `object-write` action, you do not need to specify them during invocation.

| Writing an object to a bucket with the console |
|:-----------------|
| <p><ol><li> Go to the [Actions page](https://cloud.ibm.com/functions/actions){: external} in the {{site.data.keyword.openwhisk_short}} console.</li><li> Under the `cloud-object-storage` package, click the **object-write** action.</li><li> In the Code pane, click **Change Input**.</li><li> Copy the following JSON object and replace the variables.<p><pre class="codeblock"><code>{</br>    "bucket": "bucket-name",</br>    "key": "object-name",</br>    "body": "body"</br>    "endpoint": "bucket-endpoint"</br>}</pre></code></li><li> Click **Apply**.</li><li> Click **Invoke**.</li><li> Verify the response.</li></ol></p> |
{: caption="Writing an object with the console." caption-side="top"}
{: #write-1}
{: tab-title="Console"}
{: tab-group="write"}
{: class="simple-tab-table"}

| Writing an object to a bucket with the CLI |
|:-----------------|
| <p><ol><li> Copy the following `object-write` command and replace the variables with your bucket name, object name, object body, and bucket endpoint.<p><pre class="pre"><code> ibmcloud fn action invoke /&lt;namespace&gt;/cloud-object-storage/object-write </br>--blocking --result </br>--param bucket &lt;bucket-name&gt; </br>--param key &lt;object_name&gt; </br>--param body &lt;body&gt; </br>--param endpoint &lt;bucket_endpoint&gt; </code></pre></p></li><li> Verify the output.</li></ol></p> |
{: caption="Writing an object with the CLI." caption-side="top"}
{: #write-2}
{: tab-title="CLI"}
{: tab-group="write"}
{: class="simple-tab-table"}

## Reading objects from a bucket
{: #pkg_obstorage_read}

You can use the `object-read` action to write an object to an {{site.data.keyword.cos_full_notm}} bucket. You can use this action from the console or the CLI.
{: shortdesc}

If you [bound your `bucket` and `endpoint` parameters](#pkg_obstorage_param_bind) to the `cloud-object-storage` package or to the `object-write` action, you do not need to specify them during invocation.

| Reading an object with the console |
|:-----------------|
| <p><ol><li> Go to the [Actions page](https://cloud.ibm.com/functions/actions){: external} in the {{site.data.keyword.openwhisk_short}} console.</li><li> Under the `cloud-object-storage` package, click the **object-read** action.</li><li> In the Code pane, click **Change Input**.</li><li> Copy the following JSON object and replace the variables.<p><pre class="codeblock"><code>{</br>    "bucket": "bucket-name",</br>    "key": "object_name"</br>}</pre></code></li><li> Click **Apply**.</li><li> Click **Invoke**.</li><li> Verify the response.</li><li> Check your {{site.data.keyword.cos_full_notm}} bucket for the new object.</li></ol></p> |
{: caption="Reading an object with the console." caption-side="top"}
{: #read-1}
{: tab-title="Console"}
{: tab-group="read"}
{: class="simple-tab-table"}

| Reading an object with the CLI |
|:-----------------|
| <p><ol><li> Copy the following `object-read` command and replace the variables with your bucket name and object name.<p><pre class="pre"><code> ibmcloud fn action invoke /&lt;namespace&gt;/cloud-object-storage/object-read </br>--blocking --result </br>--param bucket &lt;bucket-name&gt; </br>--param key &lt;object_name&gt;</code></pre></p></li><li> Verify the output.</li></ol></p> |
{: caption="Reading an object with the CLI." caption-side="top"}
{: #read-2}
{: tab-title="CLI"}
{: tab-group="read"}
{: class="simple-tab-table"}

### Reference
{: #pkg_obstorage_actions}

The {{site.data.keyword.cos_full_notm}} package includes the following entities:

| Entity | Type | Parameters | Description |
| --- | --- | --- | --- |
| `/cloud-object-storage` | Package | `apikey`, `cos_hmac_keys.access_key_id`, `cos_hmac_keys.secret_access_key` | Work with an {{site.data.keyword.cos_full_notm}} bucket. |
| `/cloud-object-storage/object-write` | Action | `bucket`, `key`, `body`, `endpoint`, `ibmAuthEndpoint` | Write an object to a bucket. |
| `/cloud-object-storage/object-read` | Action | `bucket`, `key`, `endpoint`, `ibmAuthEndpoint` | Read an object from a bucket. |
| `/cloud-object-storage/object-delete` | Action | `bucket`, `key`, `endpoint`, `ibmAuthEndpoint` | Delete an object from a bucket. |
| `/cloud-object-storage/bucket-cors-put` | Action | `bucket`, `corsConfig`, `endpoint`, `ibmAuthEndpoint` | Assign a CORS configuration to a bucket. |
| `/cloud-object-storage/bucket-cors-get` | Action | `bucket`, `endpoint`, `ibmAuthEndpoint` | Read the CORS configuration of a bucket. |
| `/cloud-object-storage/bucket-cors-delete` | Action | `bucket`, `endpoint`, `ibmAuthEndpoint` | Delete the CORS configuration of a bucket. |
| `/cloud-object-storage/client-get-signed-url` | Action | `bucket`, `key`, `operation`, `expires`, `endpoint` | Obtain a signed URL to restrict the Write, Read, and Delete of an object from a bucket. |

To get a full list of the available entities, run `ibmcloud fn package get cloud-object-storage`.
{: note}

#### Package parameters
{: #pkg_obstorage_pkgparams}

The following package parameters are expected to be bound to the package, and are automatically available for all actions. It is also possible to specify these parameters when you invoke one of the actions.

| Package parameter | Description |
| --- | --- |
| `apikey` | The `apikey ` parameter is IAM API key for the {{site.data.keyword.cos_full_notm}} instance. |
| `cos_hmac_keys` | The `cos_hmac_keys` parameter is the {{site.data.keyword.cos_full_notm}} instance HMAC credentials, which include the `access_key_id` and `secret_access_key` values. These credentials are used exclusively by the `client-get-signed-url` action.  Refer to [Using HMAC Credentials](/docs/cloud-object-storage/hmac?topic=cloud-object-storage-service-credentials#service-credentials) for instructions on how to generate HMAC credentials for your {{site.data.keyword.cos_full_notm}} instance. |

#### Action parameters
{: #pkg_obstorage_actparams}

The following action parameters are specified when you invoke the individual actions.  Not all of these parameters are supported by every action. Refer to the [Available entities](#pkg_obstorage_actions) table to see which parameters are supported by which action.

| Action parameter | Description |
| --- | --- |
| `bucket` | The `bucket` parameter is the name of the {{site.data.keyword.cos_full_notm}} bucket. |
| `endpoint` | The `endpoint` parameter is the {{site.data.keyword.cos_full_notm}} endpoint that is used to connect to your {{site.data.keyword.cos_full_notm}} instance. You can locate your endpoint in the [{{site.data.keyword.cos_full_notm}} documentation](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints). |
| `expires` | The `expires` parameter is the number of seconds to expire the pre-signed URL operation.  The default `expires` value is 15 minutes. |
| `ibmAuthEndpoint` | The `ibmAuthEndpoint ` parameter is the IBM Cloud authorization endpoint that is used to generate a token from the `apikey`. The default authorization endpoint works for all IBM Cloud Regions. |
| `key` | The `key` parameter is the object name. |
| `operation` | The `operation` parameter is the pre-signed URL's operation to call. |
| `corsConfig` | The `corsConfig` parameter is a bucket's CORS configuration. |
