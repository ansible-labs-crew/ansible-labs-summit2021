+++
title = "Parallel Jobs"
weight = 6
+++

The real power of instance groups is revealed when multiple jobs are started, and they are assigned to different Tower nodes. To launch parallel jobs we will set up a workflow with multiple concurrent jobs.

## Lab Scenario

To configure something meaningful we'll make a quick detour into security automation here. During this lab we’ll focus on security compliance according to STIG, CIS and so on. Often these compliance rules are enforced by executing an Ansible task per each requirement. This makes documentation and audit easier.

Compliance requirements are often grouped into independent categories. The tasks can often be executed in parallel because they do not conflict with each other.

In our demo case we use three playbooks which:

- ensure the absence of a few packages (STIG)

- ensure configuration of PAM and login cryptography (STIG)

- ensure absence of services and kernel modules (CIS).

The Playbooks can be found in the Github repository you already setup as a **Project** in your Tower.

## Prepare the Compliance Lab

### Create three Templates

As mentioned the Github repository contains three Playbooks to enforce different compliance requirements. Since you learned in the previous chapter how to do things with the AWX Collection, we want to put that knowledge to the test.

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
  - name: Compliance STIG packages Job Template
    awx.awx.tower_job_template:
      name: Compliance STIG packages
      organization: Default
      state: present
      inventory: AWX inventory
      become_enabled: True
      playbook: stig-packages.yml
      project: AWX Project
      credential: AWX Credentials
  - name: Compliance STIG config Job Template
    awx.awx.tower_job_template:
      name: Compliance STIG config
      organization: Default
      state: present
      inventory: AWX inventory
      become_enabled: True
      playbook: stig-config.yml
      project: AWX Project
      credential: AWX Credentials
  - name: Compliance CIS Job Template
    awx.awx.tower_job_template:
      name: Compliance CIS
      organization: Default
      state: present
      inventory: AWX inventory
      become_enabled: True
      playbook: cis.yml
      project: AWX Project
      credential: AWX Credentials

```

</p>
<hr/>
</details>

## Create Parallel Workflow

To enable parallel execution of the tasks in these job templates, we will create a workflow. We’ll use the web UI because using **awx** for this is a bit too involved for a lab. Workflows are configured in the **Templates** view, you might have noticed you can choose between **Job Template** and **Workflow Template** when adding a template.

- Go to the **Templates** view and click the ![plus](../../images/green_plus.png?classes=inline) button. This time choose **Workflow Template**

  - **NAME:** Compliance Workflow

  - **ORGANIZATION:** Default - click on the magnifying glass if necessary

  - Click **SAVE**

- Now the **WORKFLOW VISUALIZER** button becomes active and the     graphical workflow designer opens.

- Click on the **START** button, a new node opens. To the right you can assign an action to the node, you can choose between **TEMPLATE**, **PROJECT SYNC**, **INVENTORY SYNC** or **APPROVAL**.

- In this lab we’ll link multiple jobs to the **START**, so select the **Compliance STIG packages** job template and click **SELECT**. The node gets annotated with the name of the job.

- Click on the **START** button again, another new node opens.

- Select the **Compliance STIG config** job template and click **SELECT**. The node gets annotated with the name of the job.

- Click on the **START** button again, another new node opens.

- Select the **Compliance CIS** job template and click **SELECT**. The node gets annotated with the name of the job.

- Click **SAVE**

- In the workflow overview window, again click **SAVE**

You have configured a Workflow that is not going through templates one after the other but rather executes three templates in parallel.

![compliance workflow](../../images/compliance-workflow.png)

## Execute and Watch

Your workflow is ready to go, launch it.

- In the **Templates** view launch the **Compliance Workflow** by clicking the rocket icon.

- Wait until the workflow has finished.

Go to the **Instance Groups** view and find out how the jobs where distributed over the instances:

- Open the **INSTANCES** view of the **tower** instance group.

- Look at the **TOTAL JOBS** view of the three instances

- Because the Job Templates called in the workflow didn’t specify an instance group, they where distributed (more or less) evenly over the instances.

## Deactivate a node

Now deactivate instance **{{< param "internal_tower1" >}}** with the slider button and wait until it is shown as unavailable. Make a (mental) note of the **TOTAL JOBS** counter of the instance. Go back to the list of templates and launch the workflow **Compliance Workflow** again.

Go back to the **Instance Groups** view, get back to the instance overview of instance group **tower** and verify that the three Playbooks where launched on the remaining instances and the **TOTAL JOBS** counter of instance **{{< param "internal_tower1" >}}** didn’t change.

Activate **{{< param "internal_tower1" >}}** again by sliding the button to "checked".

## Using Instance Groups

So we have seen how a Automation Controller cluster is distributing jobs over instances by default. We have already created instance groups which allow us to take control over which job is executed on which node, so let’s use them.

To make it easier to spot where the jobs were run, let’s first empty the jobs history. This can be done using **awx-manage** on one of the Automation Controller instances. From your VSCode terminal **and as `root`** run the command:

```bash
[{{< param "control_prompt" >}} ~]$ sudo -i
[{{< param "manage_prompt" >}} ~]# awx-manage cleanup_jobs  --days=0
deleting "2020-04-08 15:43:12.121133+00:00-2-failed" (2 host summaries, 8 events)
[...]
notifications: 0 deleted, 0 skipped.
[{{< param "internal_control" >}} ~]# exit
```

### Assign Jobs to Instance Groups

One way to assign a job to an instance group is in the job template. As our compliance workflow uses three job templates, do this for all of them:

- In the web UI, go to **RESOURCES→Templates**

- Open one of the three compliance templates

- In the **Instance Groups** field, choose the **dev** instance group and click **SAVE**.

- Click **SAVE** again for the job template!

- Do this for the other two compliance templates, too.

Now the jobs that make up our **Compliance Workflow** are all configured to run on the instances of the **dev** instance group.

### Run the Workflow

You have done this a couple of times now, you should get along without detailed instructions.

- Run the **Compliance Workflow**

- What would you expect? On what instance(s) should the workflow jobs run?

- Verify\!

{{% notice tip %}}
**Result:** The workflow and the associated jobs will run on **{{< param "internal_tower2" >}}**. Okay, big surprise, in the **dev** instance group there is only one instance.
{{% /notice %}}

But what’s going to happen if you disable this instance?

- Disable the **{{< param "internal_tower2" >}}** instance in the **Instance Groups** view.

- Run the workflow again.

- What would you expect? On what instance(s) should the workflow jobs run?

- Verify\!

{{% notice tip %}}
**Result:** The workflow is running but the associated jobs will stay in pending state because there are no instance available in the **dev** instance group, and the workflow runs "forever".
{{% /notice %}}

What’s going to happen if you enable the instance again?

- Go to the **Instance Groups** view and enable **{{< param "internal_tower2" >}}** again.

- Check in the **Jobs** and **Instance Groups** view what’s happening.

{{% notice tip %}}
**Result:** After the instance is enabled again the jobs will pickup and run on **{{< param "internal_tower2" >}}**.
{{% /notice %}}

{{% notice warning %}}
At this point make sure the instances you disabled in the previous steps are definitely enabled again\! Otherwise subsequent lab tasks might fail…
{{% /notice %}}
