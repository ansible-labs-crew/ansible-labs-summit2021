+++
title = "There is more to automation controller than the Web UI"
weight = 3
+++

This is an advanced automation controller lab so we don’t really want you to use the web UI for everything. To fully embrace automation and adopt the [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) methodology, we want to use Ansible to configure our automation controller cluster.

Since automation controller is exposing all of its functionality via REST API, we can automate everything. Instead of using the API directly, it is highly recommended to use the [AWX](https://github.com/ansible/awx/tree/devel/awx_collection) or [automation controller](https://cloud.redhat.com/ansible/automation-hub/repo/published/ansible/controller) Ansible Collection (the second link will only work for you, if you have an active Red Hat Ansible Automation Platform Subscription) to setup, configure and maintain your Red Hat Ansible Automation Platform Cluster.

{{% notice info %}}
For the purpose of this lab, we will use the community AWX collection. Red Hat Customers will prefer the supported Ansible automation controller collection. Since this requires an active subscription and we want to make the lab usable for everyone, we will stick to the AWX collection for the purpose of the lab.
{{% /notice %}}

First, we need to install the AWX Collection on the `bastion` node. Installing Ansible Collections is straight forward:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ ansible-galaxy collection install awx.awx:19.2.2
```

{{% notice note %}}
The AWX collection is updated very often. To make sure the following lab instructions will work for you, we install specifically version 19.2.2. We regularly update these instruction to keep up to date and don't fall behind too much. However, to make sure the lab works for you, we recommend you use the specified version.
{{% /notice %}}

## Authentication

Before we can do any changes on our automation controller, we have to authenticate our user. There are several methods available to provide authentication details to the modules. In this lab, we'll use environment variables. In your VSCode UI use either your preferred editor from the command line or the visual editor (open a new file by going to **File⇒New File**) to open a new file and add the following:

```bash
# the Base URI of our automation controller cluster node
export CONTROLLER_HOST=https://{{< param "external_controller1" >}}

# the user name
export CONTROLLER_USERNAME=admin

# and the password
export CONTROLLER_PASSWORD='{{< param "secret_password" >}}'

# do not verify the SSL certificate, in production, you will use proper SSL certificates and not need this option or set it to True
export CONTROLLER_VERIFY_SSL=false
```

{{% notice warning %}}
As always make sure to replace `<GUID>` and `<SANDBOXID>`!
{{% /notice %}}

Save the file in the home directory of `lab-user` as `set-connection.sh`, when using the visual editor in VSCode do **File⇒Save as**.

Now set the environment variables by sourcing the file:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ source set-connection.sh
```

{{% notice warning %}}
Make sure you define the environment variables in the same shell you want to later run your Ansible Playbook from, otherwise the Playbook will fail due to authentication errors. If you lost connection to VSCode, just source the `set-connection.sh` file again.
{{% /notice %}}

## Create an inventory

Now we'll start to create a Playbook with the tasks needed to configure our automation controller. Let's start with creating an inventory and then extend the Playbook step by step.

In your VSCode UI use either your preferred editor from the command line or the visual editor again to open a new file named `configure-controller.yml` with the following content:

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
```

Since we are calling the REST API of automation controller, the Ansible Playbook is run against **localhost**, but the module will connect to the URL provided by the **CONTROLLER_HOST** environment variable you set above.

Save the file in the home directory of `lab-user` as `configure-controller.yml`.

Run your playbook:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ ansible-playbook configure-controller.yml
```

 Now verify everything worked as expected by logging into the automation controller Web UI and checking the inventory `AWX Inventory` has been created

{{% notice info %}}
You might see a warning message "You are using the awx version of this collection but connecting to Red Hat Ansible Automation Platform". This can be ignored. As said above, we intentionally use the AWX Community Collection for the purpose of the lab. As a Red Hat Customer you would probably prefer the supported Ansible Collection instead.
{{% /notice %}}

## Add hosts to inventory

Now that we have the empty inventory created, extend the Playbook to add your three managed hosts using their internal hostnames **`{{< param "internal_host1" >}}`** and **`{{< param "internal_host2" >}}`** and **`{{< param "internal_host3" >}}`**, again using the AWX Ansible Collection. The module to add hosts to an inventory is called **awx.awx.host**.

- Read the documentation and try to figure out how to add the necessary tasks to the Ansible Playbook or use the solution below.
- Add the task to the Playbook in your favorite editor.

{{% notice info %}}
Use **ansible-doc awx.awx.host** to open the documentation for the specified module.
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
      - {{< param "internal_host3" >}}
```

</p>
<hr/>
</details>

Run your playbook again and verify everything worked as expected, by logging into the automation controller Web UI and verifying the hosts have been created in the inventory `AWX Inventory`.

## Create Machine Credentials

From here onwards we'll extend the Playbook, it's up to you if you add all sections to the Playbook first and then run it or if you do it step by step.

{{% notice tip %}}
SSH keys have already been created and distributed in your lab environment and `sudo` has been setup on the managed hosts to allow password-less login. When you SSH into a host as user `lab-user` from **{{< param "internal_control" >}}** you will become user **ec2-user** on the host you logged in.
{{% /notice %}}

The next step is to configure credentials to access our managed hosts. The private key for `lab-user` is stored in `/home/lab-user/.ssh/<GUID>key.pem` and already configured on the managed nodes. Try to find the necessary attributes in the **awx.awx.credential** module documentation or use the solution provided and add this to the Playbook.

{{% notice warning %}}
And again do not forget to *replace*  **\<GUID>**! We'll stop mentioning this at some point, though... :-)
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      credential_type: Machine
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/<GUID>key.pem' ) }}"
```

</p>
<hr/>
</details>

{{% notice info %}}
If you run this Ansible Playbook multiple times, you will notice the **awx.awx.credential** module is not idempotent! Since we store the SSH key encrypted, the Ansible Module is unable to verify it has already been set and didn't change. This is what we want and expect from a secure system, but it also means Ansible has no means to verify it and hence overrides the SSH key or password every time the Ansible Playbook is executed.
Also note that the `credential_type` value is simply the type's name.
{{% /notice %}}

Run and test your playbook and verify everything works as expected, by logging into the automation controller Web UI.

## Create Project

The Ansible content used in this lab is hosted on Github in the project [https://github.com/ansible-labs-crew/playbooks_adv_summit2021.git](https://github.com/ansible-labs-crew/playbooks_adv_summit2021.git). The next step is to add a project to import the Ansible Playbooks. As before, try to figure out the necessary parameters by reading the documentation of the **awx.awx.project** module documentation.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      credential_type: Machine
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/ansible-labs-crew/playbooks_adv_summit2021.git
```

</p>
<hr/>
</details>

Run and test your playbook and verify everything works as expected, by logging into the automation controller Web UI.

If you wonder why **awx.awx.credential** is not idempotent, read the info box in the [previous chapter](#create-machine-credentials).

## Create a Template

Before running an Ansible **Job** from your automation controller cluster you must create a **Template**, again business as usual for automation controller users. For this part of the Ansible Playbook, we will use the **awx.awx.job_template** module. The name of the Ansible Playbook you want run is `apache_install.yml`.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      credential_type: Machine
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/ansible-labs-crew/playbooks_adv_summit2021.git
  - name: AWX Job Template
    awx.awx.job_template:
      name: Install Apache
      organization: Default
      state: present
      inventory: AWX inventory
      become_enabled: True
      playbook: apache_install.yml
      project: AWX Project
      credential: AWX Credentials

```

</p>
<hr/>
</details>

Run and test your playbook and verify everything works as expected, by logging into the automation controller Web UI.

## Verify the cluster

We are working in a clustered environment. To verify that the resources were created on all instances properly, login to the other controller nodes web UI's.

Have a look around, everything we automatically configured on one controller instance with our Ansible Playbook was **synchronized automatically** to the other nodes. Inventory, credentials, projects, templates, all there.

## Challenge Labs

Add a task to the Ansible Playbook you wrote in this chapter to automatically run the job **Template** you created. If you don't know how to start, check out the documentation of the **awx.awx.job_launch** module by running `ansible-doc` in your VSCode terminal:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ ansible-doc awx.awx.tower_job_launch
```

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

This is a Challenge Lab! No solution here.

</p>
<hr/>
</details>
