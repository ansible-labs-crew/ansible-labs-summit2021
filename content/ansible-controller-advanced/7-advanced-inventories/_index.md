+++
title = "Advanced Inventories"
weight = 7
+++

In Ansible and automation controller, as you know, everything starts with an inventory. There are a several methods how inventories can be created, starting from simple static definitions over importing inventory files to dynamic and smart inventories.

In real life it’s very common to deal with external dynamic inventory sources (think cloud…). In this chapter we’ll introduce you to building dynamic inventories using custom scripts. Another great feature of the automation controller to deal with inventories is the Smart Inventory feature which you’ll do a lab on as well.

## Dynamic Inventories

Quite often just using static inventories will not be enough. You might be dealing with ever-changing cloud environments or you have to get your managed systems from a CMDB or other sources of truth.

Controller includes built-in support for syncing dynamic inventory from cloud sources such as Amazon AWS, Google Compute Engine, among others. Controller also offers the ability to use custom scripts to pull from your own inventory source.

In this chapter you’ll get started with dynamic inventories in the automation controller. Aside from the build-in sources you can write inventory scripts in any programming/scripting language that you have installed on the Controller machine. To keep it easy we’ll use a most simple custom inventory script using… Bash! Yes!

{{% notice tip %}}
Don’t get this wrong… we’ve chosen to use Bash to make it as simple as possible to show the concepts behind dynamic and custom inventories. Usually you’d use Python or some other scripting/programming language.
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

Well, this is handy, the output is already configured as JSON like Ansible would expect… ;-)

{{% notice warning %}}
Okay, seriously, in real life your script would likely get some information from your source system, format it as JSON and return the data to the automation controller.
{{% /notice %}}

### The Custom Inventory Script

An inventory script has to follow some conventions. It must accept the **--list** and **--host \<hostname>** arguments. When it is called with **--list**, the script must output a JSON-encoded data containing all groups and hosts to be managed. When called with **--host \<hostname>** it must return an JSON-formatted hash or dictionary of host variables (can be empty).

As looping over all hosts and calling the script with **--host** can be pretty slow, it is possible to return a top level element called "\_meta" with all of the host variables in one script run. And this is what we’ll do. So this is our custom inventory script:

```bash
#!/bin/bash

if [ "$1" == "--list" ] ; then
    curl https://raw.githubusercontent.com/goetzrieger/ansible-labs-playbooks/master/inventory_list
elif [ "$1" == "--host" ]; then
    echo '{"_meta": {"hostvars": {}}}'
else
    echo "{ }"
fi
```

What it basically does is to return the data collected by curl when called with **--list** and as the data includes **\_meta** information about the host variables Ansible will not call it with **--host**. The curl command is of course the place where your script would get data by whatever means, format it as proper JSON and return it.

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

As simple as it gets, right? More information can be found [how to develop dynamic inventories](https://docs.ansible.com/ansible/latest/dev_guide/developing_inventory.html).

So now you have a source of (slightly static) dynamic inventory data (talk about oxymoron…) and a script to fetch and pass it to controller. Now you need to get this into controller.

### Integrate into controller

The first step is to add the inventory script to controller:

- In the web UI, open **Resources→Inventory Scripts**.

- To create a new custom inventory script, click the ![plus](../../images/green_plus.png?classes=inline) button.

- Fill in the needed data:

  - **Name:** Cloud Inventory Script

  - Copy the Bash script from above and paste it into the **CUSTOM
    SCRIPT** field

- Click **Save**

Finally the new inventory script can be used in an actual **Inventory**.

- Go to **Resources→Inventories**

- Click the ![plus](../../images/green_plus.png?classes=inline) button and choose
  **Inventory**.

- **Name:** Cloud Inventory

- Click **Save**

- The **SOURCES** button on top becomes active now, click it

- Click the ![plus](../../images/green_plus.png?classes=inline) to add a new source

- **Name:** Cloud Custom Script

- From the **SOURCE** drop-down choose **Custom Script**

- Now the dialog for the source opens, your custom script **Cloud Inventory Script** should already be selected in the **CUSTOM INVENTORY SCRIPT**.

- Under **UPDATE OPTIONS** check **Overwrite** and **Overwrite Variables**

- Click **Save**

To sync your new source into the inventory:

- Open the **Cloud Inventory** again

- Click the **SOURCES** button

- To the right click the circular arrow to start the sync process for
  your custom source.

- After the sync has finished click the **HOSTS** button (the top one).

You should now see a list of hosts according to what you got from the curl command above. Click the hosts to make sure the host variables are there, too.

## What is the take-away?

Using this simple example you have:

- Created a script to query an inventory source

- Integrated the script into controller

- Populated an inventory using the custom script

## Smart Inventories

You will most likely have inventories from different sources in your controller installation. Maybe you have a local CMDB, your virtualization management and your public cloud provider to query for managed systems. Imagine you now want to run automation jobs across these inventories on hosts matching certain search criteria.

This is where Smart Inventory comes in. A Smart Inventory is a collection of hosts defined by a stored search. Search criteria can be host attributes (like groups) or facts (such as installed software, services, hardware or whatever information Ansible pulls). A Smart Inventory can be viewed like a standard inventory and used for job runs.

The base rules of a search are:

- A search typically consists of a field (left-hand side) and a value (right-hand side)

- A colon separates the field that you want to search from the value

- A search string without a colon is treated as a simple string

### A Simple Smart Inventory

Let’s start with a simple string example. In your controller web UI, open the **Resources→Inventories** view. Then click the ![plus](../../images/green_plus.png?classes=inline) button and choose to create a new **Smart Inventory**. In the next view:

- **Name:** Smart Inventory Simple

- Click the magnifying glass icon next to **SMART HOST FILTER**

- A window **DYNAMIC HOSTS** opens, here you define the search query

To start with you can just use simple search terms. Try **cloud** or **example.com** as search terms and see what you get after hitting **ENTER**.

{{% notice tip %}}
Search terms are automatically saved so make sure to hit **CLEAR ALL** to clear the saved search when testing expressions.
{{% /notice %}}

Or what about searching by inventory groups? In the **SEARCH** field enter **`groups.name:dyngroup`**. After hitting **ENTER** the hosts from the dynamic inventory exercise should show up.

When your search returns the expected results, hit **Save** for the **DYNAMIC HOSTS** window and again for the Smart Inventory. Now your Smart Inventory is usable for executing job templates!

{{% notice tip %}}
You may press the **KEY** button to get a feeling along which fields you can search. Browsing through the API becomes necessary to understand which related fields have which attributes (e.g. name for groups).
{{% /notice %}}

### Build Smart Inventories with Facts

As you know Ansible can collect facts from managed hosts to be used in Playbooks. But before Ansible Tower 3.2 facts where only kept during a Playbook run. Ansible Tower 3.2 introduced an **integrated fact cache** to keep host facts for later usage and better performance. This is how we can use facts in searches for Smart Inventories.

#### Enable Fact Caching

{{% notice warning %}}
Fact caching is not enabled by default\!
{{% /notice %}}

Fact caching can be enabled for **Templates** and is not enabled by default. So first we have to enable it. Check **{{< param "internal_host1" >}}** and **{{< param "internal_host2" >}}** have no facts stored:

- In **Resources→Inventories** open the **Example Inventory** and click the **HOSTS** button.

- Now inspect both hosts by opening the host details and clicking the **FACTS** button at the top.

- For both hosts the **FACTS** field should be empty

Now enable fact caching for the **Install Apache** template:

- In **Resources→Templates** open the **Install Apache** template.

- Check the **ENABLE FACT CACHE** tick box and click **Save**

To gather and save the facts you have to run the job template.

- In the **Templates** list start **Install Apache** by clicking the rocket icon

Now enable fact caching for the **Remote CIS Compliance** template and run it, too. This way we’ll get cached facts for the remote DMZ host.

After you run the templates go back to the host details like you did above and check the **FACTS** fields for

- **{{< param "internal_host1" >}}** and **{{< param "internal_host2" >}}** (from the **Example Inventory**)

The hosts facts should now be populated with a lot of information.

#### Use Facts in Smart Inventory Searches

Now that we got the facts for the hosts in the facts cache, we can use facts in our searches.

- Create a new Smart Inventory named **Smart Inventory Facts**

- Open the **SMART HOST FILTER** window to enter the search

To search for facts the search field (left side of a search query) has to start with **ansible\_facts.** followed by the fact. The value is separated by a colon on the right side.

{{% notice warning %}}
No blank between field and value is allowed!
{{% /notice %}}

So what could we search for… start to look at the facts of a host. As all hosts are Red Hat, searching for the fact **ansible\_distribution:RedHat** won’t be too exciting. Ah, what the heck, just try it:

- Put **ansible\_facts.ansible\_distribution:RedHat** in the search field

- Run the search by hitting **ENTER**

There should be no surprises: All hosts you have run a fact-caching enabled template on should show up.

#### Nested Facts

A small hint: If a fact is deeper in the structure like this:

    ansible_eth0:
      active: true

The search string would look like this:
**ansible\_facts.ansible\_eth0.active:true**

### Challenge Lab: Smart Inventory with Facts

So a small challenge: Find out if all hosts have the SElinux mode set to "enforcing".

- Find the fact to use by looking at the host facts

- Create a Smart Inventory

- Create the proper search string

- Save the new Smart Inventory

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

The search string to use is:
`ansible_facts.ansible_selinux.mode:enforcing`.
It should return all hosts.

</p>
<hr/>
</details>

And to make this a bit more fun:

- SSH into one of your hosts (say **{{< param "internal_host2" >}}**) as     `ec2-user` from your VSCode terminal and set SELinux to permissive:

```bash
[{{< param "control_prompt" >}} ~]$ ssh ec2-user@{{< param "internal_host2" >}}
[ec2-user@node2 ~]$ sudo setenforce 0
```

- Run the **Install Apache** template again to update the facts.

- Make sure the host is not showing up in the Smart inventory you just created anymore.
