+++
title = "Projects & job templates"
weight = 3
+++

An Automation Controller **Project** is a logical collection of Ansible Playbooks. You manage your playbooks by placing them into a source code management (SCM) system supported by Automation Controller, including Git, Subversion, and others.

You should definitely keep your Playbooks under version control. 

## Setup a Git Repository as a Project

In this lab we’ll use existing Playbooks that are provided in this Github repository:

[https://github.com/ansible-labs-crew/playbooks_summit2021](https://github.com/ansible-labs-crew/playbooks_summit2021)

A Playbook to install the Apache webserver has already been committed to the directory **rhel/apache**, `apache_install.yml`, here for reference:

```
---
- name: Apache server installed
  hosts: all

  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest

  - name: latest firewalld version installed
    yum:
      name: firewalld
      state: latest

  - name: firewalld enabled and running
    service:
      name: firewalld
      enabled: true
      state: started

  - name: firewalld permits http service
    firewalld:
      service: http
      permanent: true
      state: enabled
      immediate: yes

  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
```

{{% notice tip %}}
Note the difference to other Playbooks you might have written\! Most importantly there is no `become` and `hosts` is set to `all`.
{{% /notice %}}

To configure and use this repository as a **Source Code Management (SCM)** system in Automation Controller you have to create a **Project** that uses the repository

## Create the Project

- Go to **Resources → Projects** in the side menu view click the ![Add](../../images/blue_add.png?classes=inline) button. Fill in the form:

- **Name:** Ansible Workshop Examples

- **Organization:** Default

- **Source Control Credential Type:** Git

Now you need the URL to access the repo. You could get the URL in Github as **Clone URL**. Enter the URL into the Project configuration:

- **Source Control URL:** `https://github.com/ansible-labs-crew/playbooks_summit2021.git`

- **Options:** Tick the boxes **Clean, Delete, Update Revision on Launch** to always get a fresh copy of the repository and to update the repository when launching a job.

- Click **Save**

The new Project will be synced automatically after creation. But you can also do this manually: Sync the Project again with the Git repository by going to the **Projects** view and clicking the circular arrow **Sync Project** icon to the right of the Project.

After starting the sync job, go to the **Jobs** view: there are new jobs for the update of the Git repository.

## Create a Job Template and Run a Job

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. So before running an Ansible **Job** from Automation Controller you must create a **Job Template** that pulls together:

- **Inventory**: On what hosts should the job run?

- **Credentials** What credentials are needed to log into the hosts?

- **Project**: Where is the Playbook?

- **What** Playbook to use?

Okay, let’s just do that: Go to the **Templates** view, click the ![Add](../../images/blue_add.png?classes=inline) button and choose **Add job template**.

{{% notice tip %}}
Remember that you can often click on magnifying glasses to get an overview of options to pick to fill in fields.
{{% /notice %}}

- **Name:** Install Apache

- **Job Type:** Run

- **Inventory:** Workshop Inventory

- **Project:** Ansible Workshop Examples

- **Execution Environment:** Controller Default EE

- **Playbook:** `rhel/apache/apache_install.yml`

- **Credentials:** Workshop Credentials

- We need to run the tasks as root so check **Enable privilege escalation** under **Options**

- Click **Save**

You can start the job by directly clicking the blue **Launch** button, or by clicking on the rocket in the Job Templates overview. After launching the Job Template, you are automatically brought to the job overview where you can follow the playbook execution in real time:

![job execution](../../images/job_overview.png)

Since this might take some time, have a closer look at all the details provided:

- The default is the **Output** view of the playbook run. Click on a node underneath a task to get detailed information for the task.

Now switch to the **Details** view:

- All details of the job template like inventory, project, credentials and playbook are shown.

- Additionally, the actual revision of the playbook is recorded here - this makes it easier to analyse job runs later on.

- Also the time of execution with start and end time is recorded, giving you an idea of how long a job execution actually was.

After the Job has finished go to the main **Jobs** view: All jobs are listed here, you should see directly before the Playbook run an SCM update was started. This is the Git update we configured for the **Project** on Job Template launch\!

## Challenge Lab: Check the Result

Time for a little challenge:

- Use an ad hoc command on all hosts to make sure Apache has been installed and is running.

You have already been through all the steps needed, so try this for yourself.

{{% notice tip %}}
What about `systemctl status httpd`?
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

- Go to **Inventories** → **Workshop Inventory**

- On the **Hosts** tab view select all hosts and click **Run Command**

- **Module:** command

- **Arguments:** systemctl status httpd

- **Next**

- **Execution Environment**: Controller Default EE

- **Next**

- **Machine Credentials:** Workshop Credentials

- Click **Launch**

</p>
<hr/>
</details>

It should show Apache running on all nodes!

## What About Some Practice?

Here is a list of tasks:

{{% notice warning %}}
Please make sure to finish these steps as the next chapter depends on it!
{{% /notice %}}

- Create a new inventory called `Webserver` and make only `node1` member of it.

- Copy the `Install Apache` template using the copy icon in the **Templates** view.

- Edit the Template and change the name to `Install Apache Ask`.

- Change the **Inventory** setting of the Project so it will prompt for the inventory on launch (tick the appropriate box).

- **Save**

- Launch the `Install Apache Ask` template.

- It will now ask for the inventory to use, choose the `Webserver` inventory and click **Next** and **Launch**.

- Wait until the Job has finished and make sure it ran only on `node1`.

{{% notice tip %}}
The Job didn’t change anything because Apache was already installed in the latest version.
{{% /notice %}}
