+++
title = "Primer on Execution Environments: Running Jobs in a Cluster"
weight = 4
+++

After boot-strapping the controller configuration from bottom up you are ready to start a job in your controller cluster. In one of your controller nodes web UI’s:

- Open the **Templates** view

- Look for the **Install Apache** Template you created with the Playbook

- Run it by clicking the rocket icon.

At first this is not different from a standard controller setup. But as this is a cluster of active controller instances each instance could have run the job.

## So, which Instance did actually run the Job?

There are a couple of ways to find the node that executed the job.

### From the Job Output

The most obvious way is to look up the **Execution Node** in the **Details** tab of the job.
If you closed it already or want to look it up later, go to **Views ⇒ Jobs** and look up the job in question.

### From the instance groups

In one of the controller instances web UI under **Administration** go to the **Instance Groups** view. For the `controlplane` instance group, the **Total Jobs** counter shows the number of finished jobs. If you click the `controlplane` instance group and switch to the **Jobs** tab in the next page, you’ll get a detailed list of jobs.

To see how jobs are distributed over the instances in an **Instance Groups**, go to the **Instances** tab under the **controlplane** group, you will get an overview of the **Total Jobs** each controller instance in this group executed.

### Using the API

- First find the job ID: In the web UI access **Views ⇒ Jobs**

- The jobs names are prefixed with the job ID, example **3 - Install Apache**

{{% notice note %}}
Make sure you choose a job with type "Playbook run".
{{% /notice %}}

- With the ID you can query the API for the instance/node the job was executed on

Bring up the terminal in your VSCode session and run:

{{% notice warning %}}
Replace **\<ID>** with the job ID you want to query and **{{< param "secret_password" >}}** with your actual password. And don't forget the proper \<GUiD>. Just saying...
{{% /notice %}}

```bash
[{{< param "pre_mng_prompt" >}} ~]$ curl -s -k -u admin:{{< param "secret_password" >}} https://{{< param "internal_controller1" >}}/api/v2/jobs/<ID>/ | python3 -m json.tool | grep execution_node

    "execution_node": "{{< param "internal_controller1" >}}",
```

{{% notice tip %}}
You can use any method you want to access the API and to display the result, of course. The usage of curl and python was just an example.
{{% /notice %}}

### Via API in the browser

Another way to query the controller API is using a browser. For example to have a look at the job details (basically what you did above using curl and friends):

{{% notice tip %}}
Note you used the internal hostname above, when using your browser, you have to use the external hostname, of course.
{{% /notice %}}

- Find the job ID

- Now get the job details via the API interface:

  - Login to the API with user `admin` and password `{{< param "secret_password" >}}`: `https://{{< param "external_controller1" >}}/api/`

  - Open the URL `https://{{< param "external_controller1" >}}/api/v2/jobs/<ID>/` where `<ID>` is the number of the job you just looked up in the UI.

  - Search the page for the string you are interested in, e.g. `execution_node`

{{% notice tip %}}
You can of course query any controller node.
{{% /notice %}}

## Ansible Execution Environments Primer!

Now that you have investigated how jobs are run in an automation controller cluster it's about time to learn about one of the major new features in Ansible Automation Platform 2: **Execution Environments**!

Before AAP 2 the Automation Platform execution relied on using **bubblewrap** to isolate processes and Python virtual environments (venv) to sandbox dependencies. This lead to a number of issues like maintaining multiple venv, migrating Ansible content between execution nodes and much more. The concept of execution environments (EE) solves this by using Linux containers.

An EE is a container run from an image that contains everything your Ansible Playbook needs to run. It's basically a control node in a box that can be executed everywhere a Linux container can run. There are ready-made images that contain everything you would expect on an Ansible control node, but you can (and probably will) start to build your own, custom image for your very own requirements at some point.

Your automation controller has been preconfigured with some standard EE images. So first go through the next section covering ad hoc commands, we'll look a bit deeper into execution environments later.

{{% notice tip %}}
Linux containers are technologies that allow you to package and isolate applications with their entire runtime environment. This makes it easy to move the contained application between environments and nodes while retaining full functionality.
In this lab you'll use the command `podman` later on. Podman is a daemon-less container engine for developing, managing, and running Open Container Initiative (OCI) containers and container images on your Linux System. If you want to learn more, there is a wealth of information on the Internet, you could start [here](http://docs.podman.io/en/latest/Introduction.html) for Podman or [here](https://www.ansible.com/blog/introduction-to-ansible-builder) for execution environments.
{{% /notice %}}

## Execution Environments: A deeper look

As promised let's look a bit deeper into execution environments. In your automation controller web UI, go to **Administration ⇒ Execution Environments**. You'll see a list of the configured execution environments and original location of the image, in our case the images are provided in the **registry.redhat.io** container registry. Here you could add your own registry with custom EE images, too.

So what happens, when automation controller runs an ad hoc command or Playbook? Let's see...

You should already have your **VS Code** terminal open in another browser tab, if not open https://{{< param "external_code" >}} and do **Terminal ⇒ New Terminal**. In this terminal:

- SSH into your automation controller node (obviously replace \<GUID> by your value):
  - `ssh autoctl1.<GUID>.internal`

- You should be the **ec2-user** user on your automation controller now, become the **awx** user by running `sudo -u awx -i`

- First let's look for the image, you should have used the **Default execution environment** in your Playbook run which should result in this image (details might be different):

```
[awx@autoctl1 ~]$ podman images
REPOSITORY                                                                         TAG     IMAGE ID      CREATED      SIZE
registry.redhat.io/ansible-automation-platform-20-early-access/ee-supported-rhel8  2.0.0   85ca2003a842  5 weeks ago  920 MB
```

- It was pulled from the registry to be available locally.

- Now we want to observe how an EE is run as a container when you execute your automation. `podman ps -w 2` will list running containers, the `-w` option updates the output regularly. Run the command, at first it will show you something like this (no container running):

```
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES
```

- Keep podman running, now it's time to execute some automation. Which should take some time so you can observe what's happening.

- In the web UI run an ad hoc command. Go to **Resources ⇒ Inventories ⇒ AWX Inventory**

- In the **Hosts** view select the three hosts.

- Click **Run Command**. Specify the ad hoc command:

  - **Module**: command

  - **Arguments**: sleep 60

  - **Next**

  - **Execution Environment**: Don't specify an EE, the **Default execution environment** will be used.

  - **Next**

  - **Machine Credential**: AWX Credentials

  - Click **Launch**

Now go back to the **VS Code** terminal. You'll see a container is launched to run the ad hoc command and then removed again:

```
CONTAINER ID  IMAGE                         COMMAND               CREATED       STATUS            PORTS   NAMES
c8e9a13ab475  quay.io/ansible/awx-ee:0.2.0  ssh-agent sh -c t...  1 second ago  Up 2 seconds ago          ansible_runner_18
```

{{% notice tip %}}
The `sleep 60` command was only used to keep the container running for some time. Your output will differ slightly.
{{% /notice %}}

- Feel free to relaunch the job again!

- Stop `podman` with `CTRL-C`.

- Logout of automation controller (twice `exit` or `CTRL-D`)

This is how automation controller uses Linux containers to run Ansible automation jobs in their own dedicated environments.
