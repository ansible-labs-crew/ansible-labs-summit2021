# Exercise 1 - Check the Prerequisites

**Read this in other languages**: ![uk](../../images/uk.png) [English](README.md),  ![japan](../../images/japan.png)[日本語](README.ja.md).

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

## Step 1.1 - Access the Environment

Login to your control host via SSH:

> **Warning**
>
> Replace **11.22.33.44** by the **IP** provided to you, and the **X** in student**X** by the student number provided to you.

    ssh studentX@11.22.33.44

> **Tip**
>
> Use the password provided on the workshops landing page.

Then become root:

    [student<X>@ansible ~]$ sudo -i

Most prerequisite tasks have already been done for you:

  - Ansible software is installed

  - `sudo` has been configured on the managed hosts to run commands that require root privileges.

Check Ansible has been installed correctly (your actual Ansible version might differ):

    [root@ansible ~]# ansible --version
    ansible 2.9.2
    [...]

> **Note**
>
> Ansible is keeping configuration management simple. Ansible requires no database or running daemons and can run easily on a laptop. On the managed hosts it needs no running agent.

Log out of the root account again:

    [root@ansible ~]# exit
    logout

> **Note**
>
> In all subsequent exercises you should work as the student&lt;X&gt; user on the control node if not explicitly told differently.

## Step 1.2 - Working the Labs

You might have guessed by now this lab is pretty command line-centric…​ :-)

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But do still take time to think and understand.

  - All labs were prepared using **Vim**, but we understand not everybody loves it. Feel free to use alternative editors. In the lab environment we provide **Midnight Commander** (just run **mc**, function keys can be reached via Esc-&lt;n&gt; or simply clicked with the mouse) or **Nano** (run **nano**). Here is a short [editor intro](../0-support-docs/editor_intro.md).

> **Tip**
>
> In the lab guide commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.

## Step 1.3 - Challenge Labs

You will soon discover that many chapters in this lab guide come with a "Challenge Lab" section. These labs are meant to give you a small task to solve using what you have learned so far. The solution of the task is shown underneath a warning sign.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
