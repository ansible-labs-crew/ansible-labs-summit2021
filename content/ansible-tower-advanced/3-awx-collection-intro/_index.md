+++
title = "There is more to Tower than the Web UI"
weight = 3
+++

This is an advanced Tower lab so we don’t really want you to use the web UI for everything. To fully embrace automation and adopt the [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) methodology, we want to use Ansible to configure our Ansible Tower Cluster.

Since Ansible Tower is exposing all of its functionality via REST API, we can automate everything. Instead of using the API directly, it is highly recommended to use the [AWX](https://github.com/ansible/awx/tree/devel/awx_collection) or [Ansible Tower](https://cloud.redhat.com/ansible/automation-hub/repo/published/ansible/tower) Ansible Collection (the second link will only work for you, if you have an active Red Hat Ansible Automation Platform Subscription) to setup, configure and maintain your Red Hat Automation Platform Cluster.

{{% notice info %}}
For the purpose of this lab, we will use the community AWX collection. Red Hat Customer will prefer the supported Ansible Tower Collection. Since this requires an active subscription and we want to make the lab usable for everyone, we will stick with the AWX collection for the purpose of the lab.
{{% /notice %}}

First, we want to install the AWX Collection. Installing Ansible Collections is super easy:

```bash
[{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install awx.awx:19.1.0
```

{{% notice note %}}
The AWX collection is updated very often. To make sure the following lab instructions will work for you, we install specifically version 19.1.0. We regularly update these instruction. to keep up to date and don't fall behind too much. However, to make sure the following instructions work for you, we suggest you to use the specified version.
{{% /notice %}}

## Authentication

Before we can do any changes on our Automation Controller, we have to authenticate our user. There are several methods available to provide authentication details to the modules. In this lab, we want to use environment variables.

```bash
# the Base URI of our Tower Cluster Node
[{{< param "control_prompt" >}} ~]$ export TOWER_HOST=https://{{< param "external_tower" >}}
# the user name
[{{< param "control_prompt" >}} ~]$ export TOWER_USERNAME=admin
# and the password
[{{< param "control_prompt" >}} ~]$ export TOWER_PASSWORD='{{< param "secret_password" >}}'
# do not verify the SSL certificate, in production, you will use proper SSL certificates and not need this option or set it to True
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

Since we are calling the REST API of Automation Controller, the Ansible Playbook is running on **localhost**, but the module will connect to the URL provided by the **TOWER_HOST** environment variable.

{{% notice info %}}
You might see a warning message "You are using the awx version of this collection but connecting to Red Hat Ansible Automation Platform". This can be ignored. As said above, we intentionally use the AWX Community Collection for the purpose of the lab. As a Red Hat Customer you would probably prefer the supported Ansible Collection instead.
{{% /notice %}}

## Add hosts to inventory

Now that we have the empty inventory created, add your two managed hosts using their internal hostnames **`{{< param "internal_host1" >}}`** and **`{{< param "internal_host2" >}}`**, again using the AWX Ansible Collection. The module to add hosts to an inventory is called **awx.awx.tower_host**. Read the documentation and try to figure out how to add the necessary tasks to the Ansible Playbook.

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
    awx.awx.tower_inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.tower_host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
```

</p>
<hr/>
</details>

## Create Machine Credentials

{{% notice tip %}}
SSH keys have already been created and distributed in your lab environment and `sudo` has been setup on the managed hosts to allow password-less login. When you SSH into a host as user **student{{< param "student" >}}** from **{{< param "internal_control" >}}** you will become user **ec2-user** on the host you logged in.
{{% /notice %}}

Now we want to configure these credentials to access our managed hosts from Automation Controller. Try to find the necessary attributes in the **awx.awx.tower_credential** module documentation.

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
    awx.awx.tower_inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.tower_host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.tower_credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
```

</p>
<hr/>
</details>

{{% notice info %}}
If you run this Ansible Playbook multiple times, you will notice the **awx.awx.tower_credential** module is not idempotent! Since we store the SSH key encrypted, the Ansible Module is unable to verify it has already been set and didn't change. This is what we want and expect from a secure system, but it also means Ansible has no means to verify it and hence overrides the password every time the Ansible Playbook is executed.
{{% /notice %}}

## Create Projects

The Ansible content used in this lab is hosted on Github. The next step is to add a project to import the Ansible Playbooks. As before, try to figure out the necessary parameters by reading the documentation of the **awx.awx.tower_project** module documentation.

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
    awx.awx.tower_inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.tower_host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.tower_credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.tower_project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/goetzrieger/ansible-labs-playbooks.git
```

</p>
<hr/>
</details>

If you wonder why **awx.awx.tower_credential** is not idempotent, read the info box in the [previous chapter](#create-machine-credentials).

## Create Job Templates

Before running an Ansible **Job** from your Automation Controller cluster you must create a **Job Template**, again business as usual for Automation Controller users. For this part of the Ansible Playbook, we will use the **awx.awx.tower_job_template** module.

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
    awx.awx.tower_inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.tower_host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.tower_credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.tower_project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/goetzrieger/ansible-labs-playbooks.git
  - name: AWX Job Template
    awx.awx.tower_job_template:
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

Now try to launch your new Job Template from the Automation Controller UI.

## Verify the cluster

We are working in a clustered environment. To verify that the resources were created on all instances properly, login to the other Tower nodes web UI's.

Have a look around, everything we automatically configured on one Tower instance with our Ansible Playbook was **synchronized automatically** to the other nodes. Inventory, credentials, projects, templates, all there.

## Challenge Labs

Try to add the necessary tasks to your Ansible Playbooks to run the **Job Template** you created.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

This is a Challenge Lab! No solution here. If you don't know where to look, check out the documentation of the **awx.awx.tower_job_template** module.

</p>
<hr/>
</details>