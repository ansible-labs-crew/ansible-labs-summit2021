+++
title = "Run a Job in a Cluster"
weight = 4
+++

After boot-strapping the controller configuration from bottom up you are ready to start a job in your controller cluster. In one of your controller nodes web UI’s:

- Open the **Templates** view

- Look for the **Install Apache** Template you created with the script

- Run it by clicking the rocket icon.

At first this is not different from a standard controller setup. But as this is a cluster of active controller instances each instance could have run the job.

## So, which Instance did actually run the Job?

There are a couple of ways to find the node that executed the job.

### From the Job Output

The most obvious way is to look up the **Execution Node** in the details of the job output (this information is only shown after the job finished). If you closed it already or want to look it up later, go to **Views->Jobs** and look up the job in question.

### From the instance groups

In one of the controller instances web UI under **Administration** go to the **Instance Groups** view. For the `controlplane` instance group, the **Total Jobs** counter shows the number of finished jobs. If you click on the instance group and then **Jobs** you’ll get a detailed list of all jobs.

To see on what instance a job actually ran go back to the **Instance Groups** view. If you click **Instances** under the **controlplane** group, you will get an overview of the **Total Jobs** each controller instance in this group executed. Clicking **Total Jobs** for an instance leads to a detailed job list for this instance.

### Using the API

- First find the job ID: In the web UI access **Views→Jobs**

- The jobs names are prefixed with the job ID, example **3 - Install Apache**

{{% notice note %}}
Make sure you choose a job with type "Playbook run".
{{% /notice %}}

- With the ID you can query the API for the instance/node the job was executed on

Bring up the terminal in your VSCode session and run:

{{% notice warning %}}
Replace **\<ID>** with the job ID you want to query and **{{< param "secret_password" >}}** with your actual password from the landing page.
{{% /notice %}}

```bash
[{{< param "pre_mng_prompt" >}} ~]$ curl -s -k -u admin:{{< param "secret_password" >}} https://{{< param "internal_controller1" >}}/api/v2/jobs/<ID>/ | python3 -m json.tool | grep execution_node

    "execution_node": "{{< param "internal_controller1" >}}",
```

{{% notice tip %}}
You can use any method you want to access the API and to display the result, of course. The usage of curl and python was just an example.
{{% /notice %}}

### Via API in the browser

Another way to query the controller API is using a browser. For example to have a look at the job details (basically what you did above using curl and friends):

{{% notice tip %}}
Note you used the internal hostname above, when using your browser, you have to use the external hostname, of course.
{{% /notice %}}

- Find the job ID

- Now get the job details via the API interface:

  - Login to the API with user `admin` and password `{{< param "secret_password" >}}`: `https://{{< param "external_controller1" >}}/api/`

  - Open the URL `https://{{< param "external_controller1" >}}/api/v2/jobs/<ID>/` where `<ID>` is the number of the job you just looked up in the UI.

  - Search the page for the string you are interested in, e.g. `execution_node`

{{% notice tip %}}
You can of course query any controller node.
{{% /notice %}}
