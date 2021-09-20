+++
title = "Role-based access control"
weight = 5
+++

You have already learned how automation controller separates credentials from users. Another advantage of automation controller is the user and group rights management.

## Automation controller Users

There are three types of automation controller users:

- **Normal User**: Have read and write access limited to the inventory and projects for which that user has been granted the appropriate roles and privileges.

- **System Auditor**: Auditors implicitly inherit the read-only capability for all objects within the automation controller environment.

- **System Administrator**: Has admin, read, and write privileges over the entire automation controller installation.

Let’s create a user:

- In the automation controller web UI menu under **Access** choose **Users**

- Click the ![add](../../images/blue_add.png?classes=inline) button

- Fill in the values for the new user:

  - **Username:** wweb

  - **Email:** wweb@example.com

  - **Password:** ansible

  - Confirm password

  - **First Name:** Werner

  - **Last Name:** Web

  - **User Type:** Normal User

- Click **Save**

## Automation controller Teams

A Team is a subdivision of an organization with associated users, projects, credentials, and permissions. Teams provide a means to implement role-based access control schemes and delegate responsibilities across organizations. For instance, permissions may be granted to a whole Team rather than each user on the Team.

Create a Team:

- Go to **Access → Teams**.

- Click the ![add](../../images/blue_add.png?classes=inline) button and create a team named `Web Content`.

Now you can add a user to the Team:

- Return to **Access -> Users** and click the wweb user.

- Jump to the **Teams** tab of user wweb.

- Click the **Associate** button and check the **Web Content** Team.

- Click **Save**

- User `wweb` is now a member of the **Web Content** Team.

## Granting Permissions

Permissions allow to read, modify, and administer projects, inventories, and other automation controller elements. Permissions can be set for different resources.

To allow users or teams to actually do something, you have to set permissions. The members of the Team **Web Content** should only be allowed to modify content of the assigned webservers.

Add the permission to use the template:

- Open the Team **Web Content**.

- Go to the **Roles** tab and click the ![add](../../images/blue_add.png?classes=inline) button.

- A new window opens. You can choose to set permissions for a number of resources.

  - Select the resource type **Job Templates**

  - Click **Next**

  - Choose the `Create index.html` Template by checking the box next to it.

  - Click **Next**

  - Choose the role **Execute**

- Click **Save**

## Test Permissions

Now log out of automation controller’s web UI and in again as the **wweb** user.

- Go to the **Templates** view, you should notice for wweb only the `Create index.html` template is listed. He is allowed to view and launch, but not to edit the Template. Just open the template and try to change it, there is not even an **Edit** button.

- Run the Job Template by clicking the rocket icon. Enter the survey content to your liking and launch the job.

- In the following **Jobs** view have a good look around, note that there where changes to the host (of course…​).

Check the result: In the **VS Code** terminal execute `curl` to pull the content of the webserver on `node1.<GUID>.internal` (you could of course check `node2.<GUID>.internal`, too):

    [{{< param "pre_mng_prompt" >}} ~]$ curl http://node1.<GUID>.internal

- In the web UI, log out user **wweb** and in again as **admin**.

Just recall what you have just done: You enabled a restricted user to run an Ansible Playbook

- Without having access to the credentials.

- Without being able to change the Playbook itself.

- But with the ability to change variables you predefined\!

Effectively you provided the power to execute automation to another user without handing out your credentials or giving the user the ability to change the automation code. And yet, at the same time the user can still modify things based on the surveys you created.

**This capability is one of the main strengths of automation controller\!**
