# Examination 7 - MariaDB installation

To make a dynamic web site, many use an SQL server to store the data for the web site.

[MariaDB](https://mariadb.org/) is an open-source relational SQL database that is good
to use for our purposes.

We can use a similar strategy as with the _nginx_ web server to install this
software onto the correct host(s). Create the playbook `07-mariadb.yml` with this content:

    ---
    - hosts: db
      become: true
      tasks:
        - name: Ensure MariaDB-server is installed.
          ansible.builtin.package:
            name: mariadb-server
            state: present

# QUESTION A

Make similar changes to this playbook that we did for the _nginx_ server, so that
the `mariadb` service starts automatically at boot, and is started when the playbook
is run.

A: We just need to add 3 tasks to complete that. state:present (install), enabled:true (ensures is starts with boot) and state:started (starts it imedietly if its not running.)

# QUESTION B

When you have run the playbook above successfully, how can you verify that the `mariadb`
service is started and is running?

A: I can SSH into dbserver and systemctl status mariadb, second option send SSH argument locally via our vagrant like:vagrant ssh dbserver -c "systemctl status mariadb". Both do kind of the same thing

# BONUS QUESTION

How many different ways can use come up with to verify that the `mariadb` service is running?

A: I think my previous methods are pretty good but after some intensive research I found this: ansible -i hosts -m shell -a "systemctl is-active mariadb", which is smoother. I dont have to SSH then exit, additionally I do not have to change folder to vagrant to run vagrant commands.

The other ones I found feels somewhat advanced for me to replicate but here is one: ansible -i hosts db -m service_facts -a "" -vvv | grep -A3 mariadb.service this one gathers via service_facts from our inventory of hosts and gives a detailed output -vvv and we then try to grep mariadb since we otherwise get tons of text.
