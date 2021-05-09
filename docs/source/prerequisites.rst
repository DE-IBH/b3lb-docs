Prerequisites
=============

.. hint::
  Some of the examples and templates are containing *Jinja2*-like variables notations ``{{ variable_name }}`` - you need to replace them appropriately if not used with a deployment or template engine like *ansible* with *Jinja2*.

Docker
------

To deploy *b3lb* you need a running Docker environment:

- Docker Engine
- Docker Swarm
- Kubernetes
- ...

This documentation expects to use *Docker Compose* for single host container deployment.

.. _Prerequisites DNS:

DNS for BBB API
---------------

The BBB API from *B3LB* can be used with a wildcard DNS entry as well as with a single domain and different URL paths. A wildcard DNS entry is recommended as it is most similar to a standalone BBB server. Both variants can be used at the same time.

Wildcard DNS Entry
__________________

*B3LB* uses the following domain scheme:

``https://api.bbbconf.de/admin/``
  the *Django Admin*

``https://api.bbbconf.de/b3lb/ping``
  checks the health of *B3LB* including database access

``https://api.bbbconf.de/b3lb/metrics``
``https://api.bbbconf.de/b3lb/stats``
  global metrics and JSON statistics

``https://tenant1.api.bbbconf.de/bigbluebutton/``
  BBB API URL for the tenant ``tentant1``

``https://tenant1-001.api.bbbconf.de/bigbluebutton/``
  BBB API URL for a additional secret for the tenant ``tentant1``

It is recommended to add corresponding DNS RR using a wildcard to your zone file::

    ; address records of reverse proxy instances
    {{ api_base_domain }}.    A       192.0.2.1
                              A       192.0.2.2
                              A       192.0.2.3
                              AAAA    2001:db8::1
                              AAAA    2001:db8::2
                              AAAA    2001:db8::3
    ; wildcard used by tenants
    *.{{ api_base_domain }}.  CNAME   api

You need to support dynamic zone updates to use wildcard certificates from *Let's'Encrypt*. The following example could be used with *bind9* to create a TSIG update key and allow zone updates.

The TSIG key can be created using the *tsig-keygen* binary:

.. code-block:: shell-session

  root@ns:~# tsig-keygen -a hmac-sha512 "{{ tsig_key }}" > /etc/bind/traefik.key

.. hint::
  - ``{{ tsig_key }}`` is the name of the TSIG key
  - ``{{ tsig_secret }}`` the secret value from the key file

Example zone definition:

.. code-block:: c

  include "/etc/bind/traefik.key";

  zone "{{ api_base_domain }}" {
      type master;
      file "{{ api_base_domain }}.zone";

      allow-update { key "{{ tsig_key }}"; };
  };

Single Domain Name
__________________

A single domain name can be used if the use of a wildcard DNS entry is not possible or not desired. The following URL patterns are used::

``https://api.bbbconf.de/admin/``
  the *Django Admin*

``https://api.bbbconf.de/b3lb/ping``
  checks the health of *B3LB* including database access

``https://api.bbbconf.de/b3lb/metrics``
``https://api.bbbconf.de/b3lb/stats``
  global metrics and JSON statistics

``https://api.bbbconf.de/b3lb/t/tenant1/bbb/``
  BBB API URL for the tenant ``tentant1``

``https://api.bbbconf.de/b3lb/t/tenant1-001/bbb/``
  BBB API URL for a additional secret for the tenant ``tentant1``


Reverse Proxy
-------------

A reverse proxy with the following features is required:

- to get a wildcard certificate from *Let's'Encrypt* the use of the `ACME DNS-01 challenge <https://letsencrypt.org/docs/challenge-types/#dns-01-challenge>`_ is required (*recommended*)
- access ACLs to protect *b3lb* admin & metrics urls

`traefik <https://github.com/traefik/traefik>`_ has proven to work very well for *b3lb*.


PostgreSQL Database
-------------------

*b3lb* requires a database backend `supported by Django <https://docs.djangoproject.com/en/3.1/ref/databases/>`_. It needs to be accessible by all *b3lb* frontend and worker instances.

.. hint::
    Using PostgreSQL 9.5+ is highly recommended.
