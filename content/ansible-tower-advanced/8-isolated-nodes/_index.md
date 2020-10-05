+++
title = "Isolated Nodes"
weight = 8
+++

Ansible is used to manage complex infrastructures with machines and networks living in multiple separate datacenters, servers behind firewalls or in cloud VPCs and remote locations only reachable over unstable links which may not survive the length of a job run. In cases like these it’s often better to run automation local to the nodes.

To solve this, Tower provides **Isolated Nodes**:

- Isolated Nodes **don’t have a full installation of Tower**, but a minimal set of utilities used to run jobs.

- Isolated Nodes can be deployed behind a firewall/VPC or in a remote datacenter, only **ingress SSH traffic** from a **controller** instance to the **isolated** instances is required.

- When a job is run that targets things managed by an isolated node, the **job** and its **environment** will be **pushed to the isolated node** over SSH.

- Periodically, the **master Ansible Tower cluster will poll the isolated node** for status on the job.

- When the **job finishes**, the job status will be **updated in Ansible Tower**.

## Prepare the Isolated Node

Because of the setup of this lab environment, before we configure the isolated node we need to do some preparation. As your student user in your VSCode terminal, execute the following steps:

```bash
    [student@ansible-1 ~]$ scp /etc/hosts isonode:
    [student@ansible-1 ~]$ ssh isonode sudo mv /home/ec2-user/hosts /etc/
```

## Setting Up Isolated Nodes

Isolated nodes are defined in the installer inventory file and setup by the Ansible Tower installer. Isolated nodes make up their own instance groups that are specified in the inventory file prefixed with **isolated\_group\_**. In the isolated instance group model, **controller** instances interact with **isolated** instances via a series of Ansible playbooks over SSH.

**So for the fun of it, let’s set one up.**

First have a look at the Tower installer inventory file that was used for lab setup. In your VSCode terminal on your Tower node 1 change into the Ansible installer directory and do the following:

    [student@ansible-1 ~]$ cd /tmp/tower_install/
    [student@ansible-1 tower_install]$ cat inventory

    [tower]
    ansible-1
    ansible-2
    ansible-3

    [database]
    ansible-4

    [...]

You can see we have the tower base group and one for the database node. For the isolated node we will define a new **isolated\_group\_** named **dmz** with one entirely new node, called **{{< param "internal_toweriso" >}}** which we’ll use to manage other hosts in the remote location.

{{% notice warning %}}
The Ansible installer files in `/tmp/tower_install/` are owned by root, but your code-server/VSCode instance is running as your student{{< param "student" >}} user. To be able to edit the inventory file, you have to change the file permissions.
{{% /notice %}}

To edit the inventory file in VSCode editor change the permissions (don't do 777 in real life... ;-)):

```bash
    [student@ansible-1 ~]$ sudo -i
    [{{< param "internal_control" >}} ~]# chmod 777 /tmp/tower_install/inventory
    [{{< param "internal_control" >}} ~]# exit
```

Then do **File -> Open File** in VSCode, navigate to `/tmp/tower_install/inventory` file and open it. Add the isolated node to the inventory to look like this:

```
    [tower]
    ansible-1
    ansible-2
    ansible-3

    [isolated_group_dmz]
    isonode ansible_host=isonode ansible_become=true

    [isolated_group_dmz:vars]
    controller=tower

    [database]
    ansible-4

    [...]
```

{{% notice warning %}}
Only add the isolated_group settings, don't change the other groups and settings! **And replace student ID and password!**
{{% /notice %}}

{{% notice tip %}}
Each isolated group must have a controller variable set. This variable points to the instance group that manages tasks that are sent to the isolated node. That instance group will be responsible for starting and monitoring jobs on the isolated node. In this case, we’re using the main tower instance group to manage this isolated group.
{{% /notice %}}

After editing the inventory, start the installer in the VSCode terminal to make the desired changes:

```bash
    [student@ansible-1 ~]$ sudo -i
    [{{< param "internal_control" >}} ~]# cd /tmp/tower_install/
    [{{< param "internal_control" >}} tower_install]# ./setup.sh
```

{{% notice note %}}
The setup.sh script will take a couple of minutes to finish execution.
{{% /notice %}}

Sit down and watch the tasks flying by...

## Verify Isolated Nodes

After the installer has finished isolated groups can be listed in the same way like instance groups and Ansible Tower cluster configuration. So the methods listed above discussing instance groups also apply to isolated nodes. For example, using `awx` as the student user:

    [student@ansible-1 ~]$ awx -f human instance_group list
    id name
    == =====
    1  tower
    2  dev
    3  prod
    4  dmz

You can see your **dmz** isolated group has been setup. Like other instance groups, isolated node groups can be assigned at the level of an organization, an inventory, or an individual job template.

## Create Isolated Node specific Inventory

To actually do something in the remote location served by the isolated node we need a managed host. Let’s assume we have a setup with one host in our DMZ, and we want to manage it siloed off from the rest of the infrastructure. The isolated node we configured above is located in the same location and is able to connect to the managed host(s).

For this you have to create a new inventory in your Tower cluster. You can do this with `awx` like we did in the beginning, or you use the web UI. Why not use the web UI for a change?

In the Tower web UI under **RESOURCES**, click **Inventories**:

- Click the ![plus](../../images/green_plus.png?classes=inline) button to add a new inventory

  - **NAME:** Remote Inventory

  - **ORGANIZATION:** Default

  - **INSTANCE GROUPS:** Pick the instance group you created in the last
    step, `dmz`

  - Click **SAVE**

Now you can add managed hosts, the **HOSTS** button is active now. Click it to access the hosts overview. There are no hosts right now, so let’s add one:

- Click the ![plus](../../images/green_plus.png?classes=inline) button to add a new host

- **NAME:** `{{< param "internal_hostremote" >}}`

- Click **SAVE**

## Create Template for Isolated Node

Next we need to assign a job template to the node. Since the node is in a DMZ, we certainly have to ensure their compliance. Thus we are going to make sure that they are following our CIS guidelines - and will set up a template executing the CIS playbook on them.

Go to **Templates** in the **RESOURCES** section of the menu, click the ![plus](../../images/green_plus.png?classes=inline) button and choose **Job Template**.

- **NAME:** Remote CIS Compliance

- **JOB TYPE:** Run

- **INVENTORY:** Remote Inventory

- **PROJECT:** Apache

- **PLAYBOOK:** `cis.yml`

- **CREDENTIAL:** Example Credentials

- **INSTANCE GROUPS:** `dmz`

- We need to run the tasks as root so check **Enable privilege escalation**

- Click **SAVE**

Next, launch the template:

- In the **Templates** view launch the **Remote CIS Compliance** job by clicking the rocket icon.

- Wait until the job has finished.

## Verify Results

Last but not least, let’s check that the job was indeed executed by the isolated node `{{< param "internal_toweriso" >}}`:

- Go to **Instance Groups** in the **ADMINISTRATION** section of the web UI

- Click on the **dmz** group.

- Click on the jobs button at the top to see the executed job.
