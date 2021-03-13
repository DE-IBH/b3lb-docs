Prerequisites
=============

Docker
------

To deploy *b3lb* you need a running Docker environment:

- Docker Engine
- Docker Swarm
- Kubernetes
- ...

.. _Prerequisites DNS:

DNS
---

*B3LB* uses the following domain scheme:

``api.bbbconf.de``
  Base domain, use for admin and global metrics access, only

``tenant1.api.bbbconf.de``
  BBB API domain for the tenant ``tentant1``

``tenant1-001.api.bbbconf.de``
  BBB API domain for a additional secret for the tenant ``tentant1``

It is recommended to add corresponding DNS RR using a wildcard to your zone file::

    ; address records of reverse proxy instances
    api         A       192.0.2.1
                A       192.0.2.2
                A       192.0.2.3
                AAAA    2001:db8::1
                AAAA    2001:db8::2
                AAAA    2001:db8::3
    ; wildcard used by tenants
    *           CNAME   api


Reverse Proxy
-------------

A reverse proxy with the following features is required:

- handle wildcard certificates using `LE DNS-01 challenge <https://letsencrypt.org/docs/challenge-types/#dns-01-challenge>`_ (*recommended*)
- restrict access to selected b3lb urls to protect admin & metrics

`traefik <https://github.com/traefik/traefik>`_ has proven to work very well for *b3lb*.


PostgreSQL Database
-------------------

*b3lb* requires a database backend `supported by Django <https://docs.djangoproject.com/en/3.1/ref/databases/>`_. It needs to be accessible by all *b3lb* frontend and worker instances.

.. hint::
    Using PostgreSQL 9.5+ is highly recommended.



Deployment
::::::::::

In 