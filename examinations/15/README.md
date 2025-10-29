# Examination 15 - Metrics (VG)

[Prometheus](https://prometheus.io/) is a powerful application used for event monitoring and alerting.

[Node Exporter](https://prometheus.io/docs/guides/node-exporter/) collects metrics for Prometheus from
the hardware and the kernel on a machine (virtual or not).

Start by running the Prometheus server and a Node Exporter in containers on your Ansible controller
(the you're running Ansible playbooks from). This can be accomplished with the [prometheus.yml](prometheus.yml)
playbook.

You may need to install [podman](https://podman.io/docs/installation) first.

If everything worked correctly, you should see the data exported from Node Exporter on http://localhost:9090/,
and you can browse this page in a web browser.

# QUESTION A

Make an Ansible playbook, `15-node_exporter.yml` that installs [Node Exporter](https://prometheus.io/download/#node_exporter)
on each of the VMs to export/expose metrics to Prometheus.

Node exporter should be running as a `systemd` service on each of the virtual machines, and
start automatically at boot.

You can find `systemd` unit files that you can use [here](https://github.com/prometheus/node_exporter/tree/master/examples/systemd), along with the requirements regarding users and permissions.

Consider the requirements carefully, and use Ansible modules to create the user, directories, copy files,
etc.

Also, consider the firewall configuration we implemented earlier, and make sure we can talk to the node
exporter port.

HINT: To get the `firewalld` service names available in `firewalld`, you can use

    $ firewall-cmd --get-services

on the `firewalld`-enabled hosts.

Note also that while running the `podman` containers on your host, you may sometimes need to stop and
start them.

    $ podman pod stop prometheus

and

    $ podman pod start prometheus

will get you on the right track, for instance if you've changed any of the Prometheus configuration.

# Resources and Information

* https://github.com/prometheus/node_exporter/tree/master/examples/systemd
* https://prometheus.io/docs/guides/node-exporter/
*
A: First I got Podman; ansible-galaxy collection install containers.podman. I added it at the beginning of Prometheus.yml
```yaml
- name: Ensure podman is installed
  ansible.builtin.package:
    name: podman
    state: present
```
So no we have Podman installed, its initiated by Prometheus.yml. I created my 15-node_exporter.yml
```yaml
---
- name: Install and enable Node Exporter
  hosts: all
  become: true
  tasks:
    - name: Create node_exporter user
      ansible.builtin.user:
        name: node_exporter
        system: true
        shell: /usr/sbin/nologin
        create_home: false

    - name: Download node_exporter
      ansible.builtin.get_url:
        url: https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
        dest: /tmp/node_exporter.tar.gz

    - name: Extract node_exporter binary
      ansible.builtin.unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /usr/local/bin/
        remote_src: true
        extra_opts: [--strip-components=1]

    - name: Create systemd service file
      ansible.builtin.copy:
        dest: /etc/systemd/system/node_exporter.service
        content: |
          [Unit]
          Description=Node Exporter
          After=network.target

          [Service]
          User=node_exporter
          ExecStart=/usr/local/bin/node_exporter
          Restart=always

          [Install]
          WantedBy=multi-user.target
        mode: "0644"

    - name: Enable and start Node Exporter service
      ansible.builtin.systemd:
        name: node_exporter
        enabled: true
        state: started

    - name: Open Node Exporter port 9100/tcp in firewalld
      ansible.posix.firewalld:
        port: 9100/tcp
        permanent: true
        immediate: true
        state: enabled

```
We create user node_exporter, install node exporter, create and run the systemd-service, make sure it start at boot, öpen port 9100. Also lifehack; use [--strip-components=1] to take away one level of extraction folder. F.eg extracts testfile123.zip directly into /temp/testfile.yml instead of temp/testfile123/testfile123.yml.

Then we can confirm its runnign by: ansible all -m shell -a "systemctl status node_exporter | grep Active".

I had to stop and rerun podman to see it popping up correctly. If I run sudo podman ps --pod I can clearly see my 3 containers. Prometheus the main server running on port 9090, node-exporter gathering the data running on port 9100 inside the pod, podman-pause managing the connection between containers. I can confirm it working via the browser on localhost:9090.
