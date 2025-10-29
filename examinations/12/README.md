# Examination 12 - Roles

So far we have been using separate playbooks and ran them whenever we wanted to make
a specific change.

With Ansible [roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) we
have the capability to organize tasks into sets, which are called roles.

These roles can then be used in a single playbook to perform the right tasks on each host.

Consider a playbook that looks like this:

    ---
    - name: Configure the web server(s) according to specs
      hosts: web
      roles:
        - webserver

    - name: Configure the database server(s) according to specs
      hosts: db
      roles:
        - dbserver

This playbook has two _plays_, each play utilizing a _role_.

This playbook is also included in this directory as [site.yml](site.yml).

Study the Ansible documentation about roles, and then start work on [QUESTION A](#question-a).

# QUESTION A

Considering the playbook above, create a role structure in your Ansible working directory
that implements the previous examinations as two separate roles; one for `webserver`
and one for `dbserver`.

Copy the `site.yml` playbook to be called `12-roles.yml`.

HINT: You can use

    $ ansible-galaxy role init [name]

to create a skeleton for a role. You won't need ALL the directories created by this,
but it gives you a starting point to fill out in case you don't want to start from scratch.


A: I added my vars file in to the 12-roles.yml that was the only issue I was having repeadetly, but when I added it that was solved. I also added become:true since its needed to modify/install packages.

As for roles I gave webserver exam 4-6, and dbserver exam 7-9. Essentially this:
roles/dbserver/tasks/main.yml
```yaml
---
- name: Ensure MariaDB-server is installed
  ansible.builtin.package:
    name: mariadb-server
    state: present

- name: Ensure python3-PyMySQL is installed
  ansible.builtin.package:
    name: python3-PyMySQL
    state: present

- name: Ensure MariaDB service is enabled and started
  ansible.builtin.service:
    name: mariadb
    enabled: true
    state: started

- name: Create database webappdb
  community.mysql.mysql_db:
    name: webappdb
    state: present
    login_unix_socket: /var/lib/mysql/mysql.sock

- name: Create MariaDB user
  community.mysql.mysql_user:
    name: webappuser
    password: "{{ webappdb_password }}"
    priv: webappdb.*:ALL
    login_unix_socket: /var/lib/mysql/mysql.sock
```

and the webserver:
```yaml
---
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present

- name: Copy nginx configuration from template
  ansible.builtin.template:
    src: templates/example.internal.conf.j2
    dest: /etc/nginx/conf.d/example.internal.conf

- name: Ensure nginx is started and enabled
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```
