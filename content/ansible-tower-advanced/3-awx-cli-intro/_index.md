+++
title = "There is more to Tower than the Web UI"
weight = 3
+++

This is an advanced Tower lab so we don’t really want you to use the web UI for everything. To fully embrace automation and adopt the infrastructure as code methodology, we want to use Ansible to configure our Ansible Tower Cluster.

Since Ansible Tower is exposing all of its functionality via REST API, we can automate everything. Instead of using the API directly, it is highly recommended to use the [AWX](https://github.com/ansible/awx/tree/devel/awx_collection) or [Ansible Tower](TODO: Add link) (this link will only work for you, if you have an active Red Hat Ansible Automation Platform Subscription) Collection to setup, configure and maintain your Ansible Tower Cluster.

{{% notice info %}}
For the purpose of this lab, we will use the community AWX collection. Red Hat Customer will prefer the supported Ansible Tower Collection. Since this requires an active subscription and we want to make the lab usable for everyone, we will stick with the AWX collection for the purpose of the lab.
{{% /notice %}}

First, we want to install the AWX Collection. Installing Ansible Collections is super easy:

```bash
[{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install awx.awx:19.1.0
```

{{% notice note %}}
The AWX collection is updated very often. To make sure the following lab instructions will work for you, we install specifically version 19.1.0. Later versions of this lab will probably use newer version of the AWX Collection.
{{% /notice %}}

## Authentication

Before we can do any changes on our Automation Controller, we have to authenticate our user. There are several methods available to provide authentication details to the modules. In this lab, we want to use environment variables.

```bash
[{{< param "control_prompt" >}} ~]$ export TOWER_HOST=https://{{< param "external_tower" >}}
[{{< param "control_prompt" >}} ~]$ export TOWER_USERNAME=admin
[{{< param "control_prompt" >}} ~]$ export TOWER_PASSWORD='{{< param "secret_password" >}}'
[{{< param "control_prompt" >}} ~]$ export TOWER_VERIFY_SSL=false
```

{{% notice warning %}}
Make sure you define the environment variables in the same shell you want to later run your Ansible Playbook from, otherwise the Playbook will fail due to authentication errors.
{{% /notice %}}

## Create an inventory

Let's start with a very simple example to see how things work.

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.tower_inventory:
      name: AWX inventory
      organization: Default
```

Since we are calling the REST API of Automation Controller, the Ansible Playbook is running on "localhost", but the module will connect to the URL provided by the "TOWER_HOST" environment variable.

{{% notice info %}}
You might see a warning message "You are using the awx version of this collection but connecting to Red Hat Ansible Automation Platform". This can be ignored. As said above, we intentionally use the AWX Community Collection for the purpose of the lab. As a Red Hat Customer you would probably prefer the supported Collection instead.
{{% /notice %}}

## Add hosts to inventory

## Create Machine Credentials

## Create Projects

## Create Job Templates

## verify the cluster

if you log into a different cluster node, you should still see the same changes.

##########################
#
# remove everythin after this line
#
##########################




After installing the tool, you have to configure authentication. The preferred way is to create a token and export it into an environment variable. After this you can seamlessly use **awx** commands in this shell. First set a number of environment variables to define your connection:

{{% notice tip %}}
Replace student number and LABID!
{{% /notice %}}

```bash
[{{< param "control_prompt" >}} ~]$ export TOWER_HOST=https://{{< param "external_tower" >}}
[{{< param "control_prompt" >}} ~]$ export TOWER_USERNAME=admin
[{{< param "control_prompt" >}} ~]$ export TOWER_PASSWORD='{{< param "secret_password" >}}'
[{{< param "control_prompt" >}} ~]$ export TOWER_VERIFY_SSL=false
```

{{% notice tip %}}
Feel free to write this into a new file using the code-server editor and then to use **source \<filename\>** to set the environment variables. This way if you loose connection to code-server you can easily re-set the vars.
{{% /notice %}}

Then use **awx** to login and print out the access token and to save it to a file at the same time:

```bash
[{{< param "control_prompt" >}} ~]$ awx login -f human | tee token
```

{{% notice tip %}}
We are saving the **export TOWER_OAUTH_TOKEN=\<YOUR_TOKEN\>** command line output to the file **token** using **tee** here to be able to set the environment variable more easily.
{{% /notice %}}

Finally set the environment variable with the token using the line the command printed out:

```bash
[{{< param "control_prompt" >}} ~]$ source token
```

Now that the access token is available in your shell, test **awx** is working. First run it without arguments to get a
list of resources you can manage with it:

    [{{< param "control_prompt" >}} ~]$ awx --help

And then test something, e.g. (leave out **-f human** if you're a JSON fan...) ;)

```bash
[{{< param "control_prompt" >}} ~]$ awx -f human user list
```

{{% notice tip %}}
When trying to find a **awx** command line for something you want to do, just move one by one.
{{% /notice %}}

Example: Need to create an inventory...

    [{{< param "control_prompt" >}} ~]$ awx --help

Okay, there is an **inventory** resource. Let’s see…

    [{{< param "control_prompt" >}} ~]$ awx inventory

Well, the **create** action sounds like what I had in mind. But what arguments do I
need? Just run:

    [{{< param "control_prompt" >}} ~]$ awx inventory create

And you'll get the required and optional arguments for the **create** action!

## Challenge Lab: awx

To practice your **awx** skills, here is a challenge:

- Try to change the **idle time out** of the Tower web UI, it’s 1800 seconds by default. Set it to, say, 7200. Using **awx**, of course.

- Start by looking for a resource type **awx** provides using **--help** that sounds like it has something to do with changing settings.

- Look at the available **awx** commands for this resource type.

- Use the commands to have a look at the parameters settings and change it.

{{% notice tip %}}
The configuration parameter is called **SESSION\_COOKIE\_AGE**
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```bash
[{{< param "control_prompt" >}} ~]$ awx setting list | grep SESSION
[{{< param "control_prompt" >}} ~]$ awx setting modify SESSION_COOKIE_AGE 7200
[{{< param "control_prompt" >}} ~]$ awx setting list | grep SESSION
```

</p>
<hr/>
</details>

If you want to, go to the web UI of any node (not just the one you connected **awx** to) and check the setting under **ADMINISTRATION→Settings→System**.
