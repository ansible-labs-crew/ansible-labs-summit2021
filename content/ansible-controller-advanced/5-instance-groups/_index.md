+++
title = "automation controller instance groups"
weight = 5
+++

Automation controller clustering allows you to easily add capacity to your controller infrastructure by adding instances. In a single-group controller cluster where all instances are within the `controlplane` group there is no way to influence which node will run a job, the cluster will take care of scheduling Jobs as it sees fit.

To enable more control over which node is running a job, Tower 3.2 saw the introduction of the **Instance Groups** feature. Instance groups allow you to organize your cluster nodes into groups. In turn Jobs can be assigned to Instance Groups by configuring the Groups in Organizations, Inventories or Job Templates.

{{% notice tip %}}
The order of priority is **Job Template > Inventory > Organization**. So Instance Groups configured in Job Templates take precedence over those configured in Inventories, which take precedence over Organizations
{{% /notice %}}

Some things to keep in mind about Instance Groups:

- Nodes in an Instance Group share a job queue.

- You can have as many Instance Groups as you like as long as there is at least one node in the `controlplane` group.

- Nodes can be in one or more Instance Groups.

- Groups can not be named `instance_group_controlplane`\!

- Controller instances can’t have the same name as a group.

Instance Groups allow some pretty cool setups, e.g. you could have some nodes shared over the whole cluster (by putting them into all groups) but then have other nodes that are dedicated to one group to reserve some capacity.

{{% notice warning %}}
The base `controlplane` group does housekeeping for all groups so the node count of this group has to scale with your overall cluster load, even if these nodes are not used to run Jobs.
Housekeeping encompasses for example processing of events from all jobs.
{{% /notice %}}

Talking about the `controlplane` group: As you have learned this group is crucial for the operations of a controller cluster. Apart from the housekeeping tasks, if a resource is not associated with any Instance Group, one of the nodes from the `controlplane` group will run the Job.
So if there are no operational nodes in the base group, the cluster will not be able to run such jobs.

{{% notice warning %}}
It is important to have enough nodes in the `controlplane` group
{{% /notice %}}

{{% notice tip %}}
There is a great [blog post](https://www.ansible.com/blog/ansible-tower-feature-spotlight-instance-groups-and-isolated-nodes) going into Instance Groups with a lot more depth.
{{% /notice %}}

## Instance Group Setup

Having the introduction out of the way, let’s get back to our lab and give Instance Groups a try.

In a basic cluster setup like ours you just have the `controlplane` base group. So let’s go and setup two instance groups:

- In **Instance Groups** add a new group by clicking the ![Add](../../images/blue_add_dd.png?classes=inline) icon and then **Add instance group**

- Name the new group **dev**

- **Save**

- Go to the **Instances** tab of the new Instances Group and add node **{{< param "internal_controller2" >}}** by clicking the **Associate** button.

Do the same to create a the new group **prod** with instance **{{< param "internal_controller3" >}}**

Go back to the **Instance Groups** view, you should now have the following setup:

- All instances are in the **controlplane** base group

- Two more groups (**prod** and **dev**) with one instances each

{{% notice tip %}}
We’re using the internal names of the controller nodes here.
{{% /notice %}}

{{% notice warning %}}
This is not best practice, it’s just for the sake of this lab! Any jobs that are launched targeting a group without active nodes will be stuck in a waiting state until instances become available. So one-instance groups are never a good idea.
{{% /notice %}}

## Verify Instance Groups

You can check your instance groups in a number of ways.

### Via the Web UI

You have configured the groups here, open the URL

    https://{{< param "external_controller1" >}}/#/instance_groups

in your browser.

In the **INSTANCE GROUPS** overview all instance groups are listed with details of the group itself like number of instances in the group, running jobs and total jobs. Like you’ve seen before the current capacity of the instance groups is shown in a live view, thus providing a quick insight if there are capacity problems.

### Via the API

You can again query the API to get this information. Either use the browser to access the URL (you might have to login to the API again):
`https://{{< param "external_controller1" >}}/api/v2/instance_groups/`

or use curl to access the API from the command line in your VSCode terminal:

```
[{{< param "pre_mng_prompt" >}} ~]$ curl -s -k -u admin:{{< param "secret_password" >}} https://{{< param "internal_controller1" >}}/api/v2/instance_groups/ | jq
```

{{% notice tip %}}
The curl command has to be on one line. Do _not_ forget or oversee the final slash at the end of the URL, it is relevant!
{{% /notice %}}

## Deactivating automation controller instances

While in the **Instances Groups** overview in the web UI click the **dev** group. In the next view change to the **Instances** tab, at the right you’ll see a slide button for each controller instance (only one in this case).

- The button should be set to **On** meaning "active". Clicking it would deactivate the corresponding instance and would prevent that further jobs are assigned to it.

- Running jobs on an instance which is set to **Off** are finished in a normal way.

- The slider **Capacity Adjustment** can change the amount of forks scheduled on an instance. This way it is possible to influence in which ratio the jobs are assigned.
