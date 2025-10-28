# Examination 5 - Handling Configuration Changes

Today, plain HTTP is considered insecure. Most public facing web sites use the encrypted HTTPS
protocol.

In order to set up our web server to use HTTPS, we need to make a configuration change in nginx.

## Preparations

Begin by running the [install-cert.yml](install-cert.yml) playbook to generate a self-signed certificate
in the correct location on the webserver.

You may need to install the Ansible `community.crypto` collection first, unless you have
already done so earlier.

In the `lab_environment` folder, there is a file called `requirements.yml` that can be used like this:

    $ ansible-galaxy collection install -r requirements.yml

Or, if you prefer, you can install the collection directly with

    $ ansible-galaxy collection install community.crypto

# HTTPS configuration in nginx

The default nginx configuration file suggests something like the following to be added to its
configuration:

    server {
        listen       443 ssl;
        http2        on;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/server.crt";
        ssl_certificate_key "/etc/pki/nginx/private/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
    }

*IMPORTANT*: If you happen to run an earlier version of
`nginx`, such as one in AlmaLinux9, it will not recognize
the "http2" directive. You can simply comment it out with
a `#`, or remove the line completely.

There are many ways to get this configuration into nginx, but we are going to copy
this as a file into `/etc/nginx/conf.d/https.conf` with Ansible with the
[ansible.builtin.copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
module.

If you have gone through the preparation part for this examinination, the certificate and the key for the
certificate has already been created so we don't need to worry about that.

In this directory, there is already a file called `files/https.conf`. Copy this directory to your Ansible
working directory, with the contents intact.

Now, we will create an Ansible playbook that copies this file via the `ansible.builtin.copy` module
to `/etc/nginx/conf.d/https.conf`.

# QUESTION A

Create a playbook, `05-web.yml` that copies the local `files/https.conf` file to `/etc/nginx/conf.d/https.conf`,
and acts ONLY on the `web` group from the inventory.

Refer to the official Ansible documentation for this, or work with a classmate to
build a valid and working playbook, preferrably that conforms to Ansible best practices.

Run the playbook with `ansible-playbook` and `--verbose` or `-v` as option:

    $ ansible-playbook -v 05-web.yml

The output from the playbook run contains something that looks suspiciously like JSON, and that contains
a number of keys and values that come from the output of the Ansible module.

What does the output look like the first time you run this playbook?

What does the output look like the second time you run this playbook?


A: THe first run, The first task(copy) succesfully executed and printed to us a  JSON like data with keys (dest, owner, mode etc) The service task then restarted and ran. the seconD run there was nothing to copy since it already exists. (If we do this after question B) we also always get chaned=1,since the service restarts nginx hence the changed=1)

# QUESTION B

Even if we have copied the configuration to the right place, we still do not have a working https service
on port 443 on the machine, which is painfully obvious if we try connecting to this port:

    $ curl -v https://192.168.121.10
    *   Trying 192.168.121.10:443...
    * connect to 192.168.121.10 port 443 from 192.168.121.1 port 56682 failed: Connection refused
    * Failed to connect to 192.168.121.10 port 443 after 0 ms: Could not connect to server
    * closing connection #0
    curl: (7) Failed to connect to 192.168.121.10 port 443 after 0 ms: Could not connect to server

The address above is just an example, and is likely different on your machine. Make sure you use the IP address
of the webserver VM on YOUR machine.

In order to make `nginx` use the new configuration by restarting the service and letting `nginx` re-read
its configuration.

On the machine itself we can do this by:

    [deploy@webserver ~]$ sudo systemctl restart nginx.service

Given what we know about the [ansible.builtin.service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html),
how can we do this through Ansible?

Add an extra task to the `05-web.yml` playbook to ensure the service is restarted after the configuration
file is installed.

When you are done, verify that `nginx` serves web pages on both TCP/80 (http) and TCP/443 (https):

    $ curl http://192.168.121.10
    $ curl --insecure https://192.168.121.10

Again, these addresses are just examples, make sure you use the IP of the actual webserver VM.

Note also that `curl` needs the `--insecure` option to establish a connection to a HTTPS server with
a self signed certificate.

A: We can add a task that does just that. name it "Restart nginx to load new cfg" to make it structured and clear. Then with ansible.builtin.service we use nginx and state: restarted. We add it last so it does all the config tasks before and then restarts it so its up to date with the latest.



# QUESTION C

What is the disadvantage of having a task that _always_ makes sure a service is restarted, even if there is
no configuration change?

A:  If you're doing this on thousands of webservers you probably get accumulated downtime which even could interrupt users currently processing or downloading something. Servers are often scheduled for server maintanance for a reason and often announced well in advance. As one of the earlier exams mentioned it also breaks the concept of "idempotency", the restarts arent really neccessary. changed=1 should only be in the logs if it actually changed something, not every time a playbook is run.

# BONUS QUESTION

There are at least two _other_ modules, in addition to the `ansible.builtin.service` module that can restart
a `systemd` service with Ansible. Which modules are they?

A: ansible.built.in.systemd_service which is almost the same as systemd service which interacts with systemd. Seems to be an older version of it also that doesnt seem to be supported(only community?) thats named community.general.systemd.
