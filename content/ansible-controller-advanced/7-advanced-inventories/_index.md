+++
title = "Advanced Inventories"
weight = 7
+++

In Ansible and automation controller, as you know, everything starts with an inventory. There are a several methods how inventories can be created, starting from simple static definitions over importing inventory files to dynamic and smart inventories.

In real life it’s very common to deal with external dynamic inventory sources (think cloud, CMDB, containers, ...). In this chapter we’ll introduce you to building dynamic inventories using custom inventory scripts. Another great feature of the automation controller to deal with inventories is the Smart Inventory feature which you’ll do a lab on as well.

## Dynamic Inventories

Quite often just using static inventories will not be enough. You might be dealing with ever-changing cloud environments or you have to get your managed systems from a CMDB or other sources of truth.

Controller includes built-in support for syncing dynamic inventory from cloud sources such as Amazon AWS, Google Compute Engine, among others. Controller also offers the ability to use custom inventory scripts to pull the data from your own inventory source.

In this chapter you’ll get started with dynamic inventories in the automation controller. Aside from the built-in sources you can write inventory scripts in any programming/scripting language that you have installed on the controller machine. To keep it easy we’ll use a most simple custom inventory script using... Bash! Yes!

{{% notice tip %}}
Don’t get this wrong... we’ve chosen to use Bash to make it as simple as possible to show the concepts behind dynamic and custom inventories. Usually you’d use Python or some other scripting/programming language.
{{% /notice %}}

### The Inventory Source

First you need a source. In **real life** this would be your cloud provider, your CMDB or what not. For the sake of this lab we put a simple file into a Github repository.

Use curl to query your external inventory source:

```bash
[{{< param "control_prompt" >}} ~]$ curl https://raw.githubusercontent.com/goetzrieger/ansible-labs-playbooks/master/inventory_list
{
    "dyngroup":{
        "hosts":[
            "cloud1.cloud.example.com",
            "cloud2.cloud.example.com"
        ],
        "vars":{
            "var1": true
        }
    },
    "_meta":{
        "hostvars":{
            "cloud1.cloud.example.com":{
                "type":"web"
            },
            "cloud2.cloud.example.com":{
                "type":"database"
            }
        }
    }
}
```

Well, this is handy, the output is already presented as JSON, the way Ansible would expect... ;-)

{{% notice warning %}}
Okay, seriously, in real life your script would likely get some information from your source system, format it as JSON and return the data to the automation controller.
{{% /notice %}}

### The Custom Inventory Script

An inventory script has to follow some conventions. It must accept the **--list** and **--host \<hostname>** arguments. When it is called with **--list**, the script must output a JSON-encoded data containing all groups and hosts to be managed. When called with **--host \<hostname>** it must return an JSON-formatted hash or dictionary of host variables (can be empty).

As looping over all hosts and calling the script with **--host** can be pretty slow, it is possible to return a top level element called "\_meta" with all of the host variables in one script run. And this is what we’ll do. So this is our custom inventory script:

```bash
#!/bin/bash

if [ "$1" == "--list" ] ; then
    curl -sS https://raw.githubusercontent.com/goetzrieger/ansible-labs-playbooks/master/inventory_list
elif [ "$1" == "--host" ]; then
    echo '{"_meta": {"hostvars": {}}}'
else
    echo "{ }"
fi
```

What it basically does is to return the data collected by curl when called with **--list** and as the data includes **\_meta** information about the host variables Ansible will not call it with **--host**. The curl command is of course the place where your script would get data by whatever means, format it as proper JSON and return it (`-sS` makes curl silent, except for error messages).

But before we integrate the custom inventory script into our controller cluster, it’s a good idea to test it on the command line first:

- Bring up your VSCode terminal
- Create the file `dyninv.sh` with the content shown above (use VI or the VSCode editor)
- Make the script executable:

```bash
[{{< param "control_prompt" >}} ~]$ chmod +x dyninv.sh
```

- Execute it:

```bash
[{{< param "control_prompt" >}} ~]$ ./dyninv.sh --list
{
    "dyngroup":{
        "hosts":[
            "cloud1.cloud.example.com",
            "cloud2.cloud.example.com"
        ],
        "vars":{
            "var1": true
        }
    },
    "_meta":{
        "hostvars":{
            "cloud1.cloud.example.com":{
                "type":"web"
            },
            "cloud2.cloud.example.com":{
                "type":"database"
            }
        }
    }
}
```

The script should output the JSON-formatted output shown above.

As simple as it gets, right? More information can be found on [how to develop dynamic inventories](https://docs.ansible.com/ansible/latest/dev_guide/developing_inventory.html).

So now you have a source of (slightly static) dynamic inventory data (talk about oxymoron…) and a script to fetch and pass it to controller. Now you need to get this into controller.

### Integrate into controller

The first step is to add the inventory script to controller is to create the git repository where the inventory script is stored.

- In the web UI, open **Resources -> Projects**.

- Add a new project by clicking on the ![add](../../images/blue_add.png?classes=inline) button

- Enter the necessary information:

  - **Name:** Dynamic Inventory

  - **Default Execution Environment:** Default execution environment

  - **Source Control Credential Type:** Git

  - **Source Control URL:** https://github.com/ansible-labs-crew/playbooks_summit2021.git

  - enable **Update Revision on Launch**

- Click **Save**.

You can watch the initial project sync by navigating to **Views -> Jobs**. Wait until it completed successfully, before you continue.

The second step is to add the dynamic inventory and point it to the Git project you just added.

- In the web UI, open **Resources→Inventories**.

- To create a new custom inventory, click the ![add](../../images/blue_add.png?classes=inline) button and click on **Add inventory**.

- Fill in the needed data:

  - **Name:** Cloud Inventory

- Click **Save**

- In the Details page, click on **Sources** and once more on the blue ![add](../../images/blue_add.png?classes=inline) button.

- Fill in the needed data:

  - **Name:** Cloud Inventory Script

  - **Source:** Sourced from a project

  - **Project:** Dynamic Inventory

  - **Inventory file:** inventory/inventory-script

  - enable **Update on launch**

- Click on **Save**

- Start the initial sync by clicking on **Sync**

Navigate to **Views -> Jobs** to watch the initial sync. When you click on the job, you should see the script running and the inventory data reported by the script.

Investigate the new hosts which were added to your inventory, by navigating to **Resources -> Hosts**. You should find two new hosts: `cloud1.cloud.example.com` and `cloud2.cloud.example.com`.

### What is the take-away?

Using this simple example you have:

- Created a script to query an inventory source

- Integrated the script into controller

- Populated an inventory using the custom script

## Smart Inventories

You will most likely have inventories from different sources in your controller installation. Maybe you have a local CMDB, your virtualization management and your public cloud provider to query for managed systems. Imagine you now want to run automation jobs across these inventories on hosts matching certain search criteria.

This is where Smart Inventories come in. A Smart Inventory is a collection of hosts defined by a stored search. Search criteria can be host attributes (like groups) or facts (such as installed software, services, hardware or whatever information Ansible facts are collected). A Smart Inventory can be viewed like a standard inventory and used for job runs.

Automation controller 4.0 introduces a new UI to build these search filters without the need of advanced regular expression kung-fu.

### A Simple Smart Inventory

Let’s start with a simple string example. In your controller web UI, open the **Resources -> Inventories** view. Then click the ![add](../../images/blue_add.png?classes=inline) button and choose to create a new **Smart Inventory**. In the next view:

- **Name:** Simple Smart Inventory

- Click the magnifying glass icon next to **Smart host filter**

- A window **Perform a search to define a host filter** opens, here you define the search query

To start with you can just use simple search terms. Try **cloud** or **example.com** as search terms and see what you get after hitting **ENTER**.

{{% notice tip %}}
Search terms are automatically saved so make sure to hit **Clear all filters** to clear the saved search when testing expressions.
{{% /notice %}}

Or what about searching by inventory groups? Switch from **Name** to **Group** and enter `dyngroup` into the search field. After hitting **ENTER** you should only see cloud1 and cloud2.

When your search returns the expected results, hit **Select** for the **Perform a search to define a host filter** window and **Save** for the Smart Inventory. Now your Smart Inventory is usable for executing job templates!

{{% notice tip %}}
There are many additional attributes you can create a filter for - including Ansible facts returned from your managed nodes - but that's for another lab...
{{% /notice %}}

### Advanced lab

Create a filter to find only enabled hosts in your smart inventory. Hosts can be temporarily disabled, for example due to some maintenance work. We want to exclude them from our inventory.
