# Examination 13 - Handlers

In [Examination 5](../05/) we asked the question what the disadvantage is of restarting
a service every time a task is run, whether or not it's actually needed.

In order to minimize the amount of restarts and to enable a complex configuration to run
through all its steps before reloading or restarting anything, we can trigger a _handler_
to be run once when there is a notification of change.

Read up on [Ansible handlers](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)

In the previous examination ([Examination 12](../12/)), we changed the structure of the project to two separate
roles, `webserver` and `dbserver`.

# QUESTION A

Make the necessary changes to the `webserver` role, so that `nginx` only reloads when it's configuration
has changed in a task, such as when we have changed a `server` stanza.

Also note the difference between `restarted` and `reloaded` in the [ansible.builtin.service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html) module.

In order for `nginx` to pick up any configuration changes, it's enough to do a `reload` instead of
a full `restart`.

A: Most essential difference is I added "notify: reload nginx" which will trigger the handler file and trigger the reload to each task that should require it. No longer using "when:". This feels much more inutiative and less destructive. I also combined tasks such as Mariadb & PyMySQL state checks into 1 task. I also added minor stuff I thought was missing. HEre is the latest:

```yaml
- name: Deploy https.conf
  ansible.builtin.copy:
    src: https.conf
    dest: /etc/nginx/conf.d/https.conf
    owner: root
    group: root
    mode: "0644"
  notify: Reload nginx

- name: Deploy vhost template
  ansible.builtin.template:
    src: example.internal.conf.j2
    dest: /etc/nginx/conf.d/example.internal.conf
    owner: root
    group: root
    mode: "0644"
  notify: Reload nginx
```
