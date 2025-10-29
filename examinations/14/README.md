# Examination 14 - Firewalls (VG)

The IT security team has noticed that we do not have any firewalls enabled on the servers,
and thus ITSEC surmises that the servers are vulnerable to intruders and malware.

As a first step to appeasing them, we will install and enable `firewalld` and
enable the services needed for connecting to the web server(s) and the database server(s).

# QUESTION A

Create a playbook `14-firewall.yml` that utilizes the [ansible.posix.firewalld](https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html) module to enable the following services in firewalld:

* On the webserver(s), `http` and `https`
* On the database servers(s), the `mysql`

You will need to install `firewalld` and `python3-firewall`, and you will need to enable
the `firewalld` service and have it running on all servers.

When the playbook is run, you should be able to do the following on each of the
servers:

## dbserver

    [deploy@dbserver ~]$ sudo cat /etc/firewalld/zones/public.xml
    <?xml version="1.0" encoding="utf-8"?>
    <zone>
      <short>Public</short>
      <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
      <service name="ssh"/>
      <service name="dhcpv6-client"/>
      <service name="cockpit"/>
      <service name="mysql"/>
    <forward/>
    </zone>

## webserver

    [deploy@webserver ~]$ sudo cat /etc/firewalld/zones/public.xml
    <?xml version="1.0" encoding="utf-8"?>
    <zone>
      <short>Public</short>
      <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
      <service name="ssh"/>
      <service name="dhcpv6-client"/>
      <service name="cockpit"/>
      <service name="https"/>
      <service name="http"/>
      <forward/>
    </zone>

# Resources and Documentation

https://firewalld.org

A: Very cool to see so much progress, the servers are shaping up and Im starting to feel like a doer. This time around I tried using handler in the config file directly instead of calling it externally. I installed http/https on the web and mysql on the dbserver. My reload handler will trigger as soon as a noteable change is made. Im also starting to make the playbooks a bit more clean and tidier than before.
