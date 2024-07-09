# 🧪 How to Install Ansible: A Step-by-Step Guide

![alt text](image.png)

## Description

Looking to automate your IT infrastructure and streamline your operations? This comprehensive guide will walk you through the process of installing Ansible, enabling you to manage your systems more efficiently and effectively.

## Steps 👓:-

**Step 1** — Update /etc/hosts

You can configure the BIND DNS server to resolve the hostname, or alternatively, update the /etc/hosts file with the hostname and IP details of your controller and managed hosts in your setup.

```
$ cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.56.112 localhost.localdomain
192.168.56.122 master1
192.168.56.123 worker1
```

**Step 2** — Install Ansible

**2A:** Method 1 — Install Ansible using EPEL repo

```
$ dnf -y install epel-release
```

Now once epel repo is installed you can search for ansible package

```
$ dnf search ansible
=========================================================== Name & Summary Matched: ansible ===========================================================
ansible.noarch : Curated set of Ansible collections included in addition to ansible-core
```

So you can now install ansible.noarch rpm on the controller node using dnf or yum

```
$ dnf install -y ansible.noarch
```

**2B:** Method 2 — Install Ansible

```
$ dnf install ansible -y
```

Based on your environment, once you install Ansible, you will notice that Python 3 is also installed as a dependency.

```
$ ansible --version
```

Sample Output:

```
[root@localhost ~]$ ansible --version
ansible [core 2.14.17]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /home/root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.18 (main, Jan 24 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

**Step 3** — Create normal user

This is important because we will use this user to perform all Ansible-related tasks. For the sake of this article, I will create a user named "ansible."

```
$ useradd ansible
```

Assign a password to this user.

```
$ passwd ansible
```

Ensure you follow the same steps on the managed nodes by creating the identical user on each of your managed hosts.

For example, in master1:
```
$ useradd ansible
$ passwd ansible
```

**Step 4** — Generate SSH keys and distribute them to the managed nodes

Now, we need to enable passwordless login between our controller node and all the managed hosts. This involves configuring passphrase-based login using ssh-keygen.

Please log in or switch to the "ansible" user and execute ssh-keygen with the following format. Use -P to set a null password for the key pair.

```
$ ssh-keygen -t rsa -P ""
```

For example:
```
ansible@master1:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ansible/.ssh/id_rsa):
Created directory '/home/ansible/.ssh'.
Your identification has been saved in /home/ansible/.ssh/id_rsa
Your public key has been saved in /home/ansible/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:B5jhkCR/7hSbCdRC9wJ9Hln+b7o8UpVX3Y0FV6pB+8g ansible@master1
The key's randomart image is:
+---[RSA 3072]----+
|  .o*+o  o. . .+O|
|   +o=o=+. . ..o=|
|    o.Boo.. o ...|
|     + *.. o =o .|
|      * S . E... |
|     o   .  ..   |
|      .    .  o  |
|          ...o   |
|           .+o   |
+----[SHA256]-----+
```

This action will generate a public and private key pair in the home directory under ~/.ssh/. With these keys ready, you should copy the public key to the target managed server. Utilize ssh-copy-id for efficiency, as it automates the necessary steps to enable passphrase-based login.

```
$ ssh-copy-id master1
```

Perform the same action for the remaining managed hosts.

Additionally, copy the public key for localhost from the controller node, as it will be needed later.

**Step 5** — Set up privilege escalation using sudo

To grant privilege escalation to our Ansible user, we will create a new sudoers rule using a file under /etc/sudoers.d

```
$ cat /etc/sudoers.d/ansible
```

Apply the same rule to all your managed hosts.

For example, master1:
```
$ echo "ansible ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/ansible
```

**Step 6** — Create an inventory file

Create an inventory file named inventory.ini (inventory name is any)

```
$ vi inventory.ini 
```

Add the following into the inventory file.
```
[machines]
<MANAGED-HOST-IP-ADDRESS>
```

**Step 7** — Test a ping

```
ansible all -m ping -i inventory.ini
```

Sample Result.
```
192.168.56.122 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Final Note

If you find this repository useful for learning, please give it a star on GitHub. Thank you!

**Authored by:** [ELemenoppee](https://github.com/ELemenoppee)