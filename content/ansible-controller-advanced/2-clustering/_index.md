+++
title = "Introduction to Automation Controller Clustering"
weight = 2
+++

With version 3.1 Ansible Tower introduced clustering, replacing the
redundancy solution configured with active-passive nodes.
Clustering is sharing load between controller nodes/instances.
Each controller instance is able to act as an entry point for UI and API access.

{{% notice tip %}}
Using a load balancer in front of the controller nodes is possible, but optional because an automation controller cluster can be accessed via all controller instances.
{{% /notice %}}

Each instance in a controller cluster expands the cluster’s capacity to
execute jobs. Jobs can and will run anywhere rather than be directed on
where to run.

{{% notice tip %}}
The Appendix contains some installation considerations and an installer inventory for reference.
{{% /notice %}}

## Access the Controller Web UI

For the first contact to your cluster open your browser and login to the
controller node 1 web UIs as:

- user **admin**

- password **{{< param "secret_password" >}}**

{{% notice warning %}}
Use the pre-created URLs from the lab landing page or replace **{{< param "labid" >}}** with the session ID and **{{< param "student" >}}** with your student number!
{{% /notice %}}

**`https://{{< param "external_controller1" >}}`**

Just from the web UI you wouldn’t know you’ve got a controller cluster at your hands here. To learn more about your cluster and its state, in one of the instances web UI under **ADMINISTRATION** choose **Instance Groups**. Here you will get an overview of the cluster by instance groups. Explore the information provided, of course there is no capacity used yet and no Jobs have run.

Right now we have only one instance group named **controlplane**. When you get more groups, from this view you will see how the instance are distributed over the groups.

To dig deeper click on **INSTANCES** to get more information about the instances allocated to a group. In the instances view you can toggle nodes off/online and adjust the number of forks (don't do this now).

You’ll learn more about this later.

## Access your controller cluster via command line

You can also get information about your cluster on the command line. Log in to your **VS Code** again if you closed it by opening this URL in your browser:

**`https://{{< param "external_code" >}}`**

Your VSCode session is running on your controller node 1. Again if not still open, open a terminal by clicking **Terminal->New Terminal** in the menu bar.

A terminal window opens at the bottom, become root:

```bash
    [{{< param "pre_mng_prompt" >}} ~]$ sudo -i
    [{{< param "manage_prompt" >}} ~]#
```

In the terminal run the following command:

> Your exact hostnames will differ, of course!

```bash
    [{{< param "manage_prompt" >}} ~]# awx-manage list_instances
    [controlplane capacity=51]
        {{< param "internal_controller1" >}} capacity=57 version=4.0.0 heartbeat="2020-08-27 09:06:21"
        {{< param "internal_controller2" >}} capacity=57 version=4.0.0 heartbeat="2020-08-27 09:05:58"
        {{< param "internal_controller3" >}} capacity=57 version=4.0.0 heartbeat="2020-08-27 09:06:00"
```

So what we’ve got is a three-node controller cluster, no surprises here. In addition the command tells us the capacity (maximum number of forks/concurrent jobs) per node and for the instance groups. Here the capacity value of 57 is allocated to any of our three nodes.

{{% notice tip %}}
The **awx-manage** (formerly tower-manage) utility can be used to manage a lot of the more internal aspects of the automation controller.
You can e.g. use it to clean up old data, for token and session management and for cluster management.
{{% /notice %}}
