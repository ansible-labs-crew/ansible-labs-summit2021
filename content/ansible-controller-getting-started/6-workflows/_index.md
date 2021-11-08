+++
title = "Workflows"
weight = 6
+++

The basic idea of an automation controller workflow is to link multiple Job Templates together. They may or may not share inventory, Playbooks or even permissions. The links can be conditional, e.g.:

- If job template A succeeds, job template B is automatically executed afterwards.

- In case of a failure, job template C will be run.

And the workflows are not even limited to Job Templates, but can also include project or inventory updates.

This enables new applications for automation controller: different Job Templates can build upon each other. E.g. the networking team creates playbooks with their own content, in their own Git repository and even targeting their own inventory, while the operations team also has their own repos, playbooks and inventory.

In this lab you’ll learn how to setup a workflow.

## Lab Scenario

You have two departments in your organization:

- The web operations team that is developing Playbooks for deploying web infrastructures in their own Git repository.

- The web applications team, that develops JavaScript web applications for NodeJS in their Git repository.

When there is a new NodeJS-based website to deploy, two main steps need to happen:

The web operations team has to:

- Install and configure NodeJS to run as a service.

- An Apache instance needs to be installed and configured as proxy to pass requests for the NodeJS content to the NodeJS backend. And a lot of other steps might be needed, too. Like SELinux, firewall... you know the drill.

The web developer team has to:

- Deploy the most recent version of the JavaScript web application.

- Make sure everything is prepared to run the application like the directory structure and service restarts.

To make things somewhat easier for you, everything needed already exists in a Github repository: Playbooks, JavaScript files etc. You just need to glue it together.

## Set up Projects

First you have to set up the Git repo as Projects like you normally would. You have done this before, try to do this on your own. Detailed instructions can be found below.

{{% notice warning %}}
If you are still logged in as user **wweb**, log out and log in as user **admin** again.
{{% /notice %}}

- Create the project for web operations:

  - It should be named **Webops Git Repo**.

  - The URL to access the repo is **https\://github.com/ansible-labs-crew/playbooks_ops_summit2021.git**

- Create the project for the application developers:

  - It should be named **Webdev Git Repo**.

  - The URL to access the repo is **https\://github.com/ansible-labs-crew/playbooks_dev_summit2021.git**

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

- Create the project for web operations. In the **Projects** view click the blue **Add** button and fill in:

  - **Name:** Webops Git Repo

  - **Organization:** Default

  - **Source Control Credential Type:** Git

  - **Source Control URL:** https\://github.com/ansible-labs-crew/playbooks_ops_summit2021.git

   - **Options:** Clean, Delete, Update Revision on Launch

- Click **Save**

- Create the project for the application developers. In the **Projects** view click the blue **Add** button and fill in:

  - **Name:** Webdev Git Repo

  - **Organization:** Default

  - **Source Control Credential Type:** Git

  - **Source Control URL:** https\://github.com/ansible-labs-crew/playbooks_dev_summit2021.git

  - **Options:** Clean, Delete, Update Revision on Launch

- Click **Save**

</p>
<hr/>
</details>

## Set up Job Templates

Now you have to create Job Templates like you would for "normal" Jobs.

{{% notice tip %}}
We want to install the **NodeJS app on node3 only**. The Inventory **Workshop Inventory** contains all nodes, but we can limit the nodes using the **LIMIT** field!
{{% /notice %}}

- Go to the **Templates** view, click the ![Add](../../images/blue_add.png?classes=inline) and choose **Add job template**:

  - **Name:** Web Infra Deploy

  - **Job Type:** Run

  - **Inventory:** Workshop Inventory

  - **Project:** Webops Git Repo

  - **Execution Environment:** Default execution environment

  - **Playbook:** `web_infrastructure.yml`

  - **Credentials:** Workshop Credentials

  - **Limit:** node3.\<GUID>.internal

  - **Options:** Privilege Escalation

- Click **Save**

- Go to the **Templates** view again, ![Add](../../images/blue_add.png?classes=inline) and choose **Add job template**:

  - **Name:** Web App Deploy

  - **Job Type:** Run

  - **Inventory:** Workshop Inventory

  - **Project:** Webdev Git Repo

  - **Execution Environment:** Default execution environment

  - **Playbook:** `install_node_app.yml`

  - **Credentials:** Workshop Credentials

  - **Limit:** node3.\<GUID>.internal

  - **OPTIONS:** Privilege Escalation

- Click **Save**

{{% notice tip %}}
If you want to know what the Playbooks look like, check out the Github URL.
{{% /notice %}}

## Set up the Workflow

And now you finally set up the Workflow. Workflows are configured in the **Templates** view, you might have noticed you can choose between **Add job template** and **Add workflow template** when adding a template so this is finally making sense.

- Go to the **Templates** view and click the ![Add](../../images/blue_add.png?classes=inline) button. This time choose **Add workflow template**

  - **Name:** Deploy Webapplication

  - **Organization:** Default

- Click **Save**

- After saving the template the **Workflow Visualizer** opens to allow you to build a workflow. You can later open the **Workflow Visualizer** again by using the button on the template details page.

- Click on the **Start** button, a dialog to configure a new **workflow node** opens.

- First select the node type, you can choose between **Job Template**, **Project Sync**, **Inventory Source Sync**, **Approval** and **Workflow Job Template**.

- Select **Job Template**

- Choose the **Web Infra Deploy** Template, you might have to switch pages.

- Click **Save**.

You get into the **Workflow Visualizer** overview showing the first workflow node.

- The node gets annotated with the name of the job.

- Hover the mouse pointer over the node, you’ll see a number of options appearing:

  - **+** to add a new workflow node

  - **i** for Job Template details

  - a pencil icon to edit this node's settings

  - a link icon to link to another node

  - and a delete icon.

- We want to add another workflow node, click the **+** sign

- You could now set the condition under which the next node is executed, select **On Success**

- Click **Next**

{{% notice tip %}}
The type allows for more complex workflows. You could lay out different execution paths for successful and for failed Playbook runs.
{{% /notice %}}

- Choose **Web App Deploy** as the next Job.

- Click **Save**

- Back in the main **Workflow Visualizer** view click **Save** in the upper right to save the Workflow Template.

{{% notice tip %}}
The **Workflow Visualizer** has options for setting up more advanced workflows, please refer to the [documentation](https://docs.ansible.com/ansible-tower/latest/html/userguide/workflows.html).
{{% /notice %}}

## And Action

Your **Deploy Webapplication** workflow is ready to go, launch it.

- Click the **Launch** button directly or go to the the **Templates** view and launch the **Deploy Webapplication** workflow by clicking the rocket icon.

![jobs view of workflow](../../images/job_workflow.png)

Note how the workflow run is shown in the job view as a visual representation of the different workflow steps. Same as for a normal job template executions you can go to the **Details** tab to get more information. If you want to look at the actual Jobs behind the workflow nodes, click the workflow nodes. If you want to get back from a details view to the corresponding workflow, just hit your browsers back button.

After the job has finished, check if everything worked fine. In your **VS Code** terminal, run:

    [{{< param "pre_mng_prompt" >}} ~]$ curl http://node3.<GUID>.internal/nodejs

You should be greeted with a friendly `Hello World`

## Workflow Approvals

To make Workflows in automation controller even more useful you can add approval steps in your Workflow. Using this feature it's possible to have another user review and approve all or certain nodes in a Workflow. In addition you could set time limits, so a Workflow waiting for approval would be cancelled or proceed after time runs out automatically.

Let's see how this works by adding an approval step to the **Deploy Webapplication** Workflow Template you configured above. The goal is to:

- Allow user **wweb** to execute the workflow by allowing the team **Web Content** access.

- To have a user do the approval.

- Require an approval before the application is installed.

Start with inserting the approval into the workflow:

- As user **admin** open the Workflow by going to **Resources ⇒ Templates ⇒ Deploy Webapplication**.

- Click  **Visualizer** to open the Workflow editor.

- Hover the mouse over the green connection line between the to nodes and click **+** to add another node.

- As run condition choose **Always**.

- Click **Next**

- Set **Node Type** to **Approval**

- **Name**: approve app install

- **Save** and **Save** again for the workflow.

![Workflow Approval](../../images/workflow_approval.png)

Now give members of the team **Web Content** access rights to execute the Workflow. You did this in the RBAC excercise, basically do this:

- Go to the **Access** tab of the Workflow.

- Click ![Add](../../images/blue_add.png?classes=inline) and add team **Web Content**.

- Select the role **Execute**

- **Save**

Log out and log in again as user **wweb** with password `ansible`.

- You should be able to execute the Workflow Template **Deploy Webapplication**

- Launch it

- The workflow should be stuck in the **approve app install** step, waiting for approval.

- In the upper right corner you'll see a notification **1** near to the bell icon. Click it.

![Approval Notification](../../images/approval_notification.png)

- You'll get to the **Workflow Approvals** view listing pending approvals.
- If you check the box to the right of the approval, you'll note you **can't approve** because the **Approve** and **Deny** buttons stay inactive.

No surprise, you don't have the permissions as user **wweb**. Log in as admin again and create the approval user (you did this before in the RBAC exercise):

- **Username:** aapproval

- **Password:** ansible

- **First Name:** Anton

- **Last name:** Approval

- **User Type:** Normal User

- **Save**

{{% notice tip %}}
You could of course approve the workflow as user **admin**, but we'd like to create a dedicated user for this!
{{% /notice %}}

Now go back to the Workflow **Deploy Webapplication**

- Open the **Access** tab

- Add a **aapproval** as new user

- Give the role **Approve**

- **Save**

You're set. The Workflow run **wweb** started is still in pending state, you can check this by going to **Views ⇒ Jobs**, it will still be shown as running. Okay, let's approve it!

- Log out and in as user **aapproval**

- You'll again see the bell notification in the upper right, click it to get to the **Workflow Approvals** view.

- Check the box to the left of the **approve app install** job.

- You should now see the **Approve** and **Deny** buttons getting active!

- Click the **Approve** button.

- Go to **Views ⇒ Jobs** and click the **Deploy Webapplicattion** job.

You'll see how the job picks up after you approved it and goes to the **Web App Deploy** node to finish.

This was a basic example to show how you could create a Job, allow one user to execute it and another user to approve it!

{{% notice tip %}}
Make sure you log out as user `aapproval` and in again as user `admin` before you proceed!
{{% /notice %}}