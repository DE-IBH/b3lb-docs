About
=====

B3LB is based on the Django_ Python Web framework and is designed to work in large scale-out deployments with 100+ BigBlueButton nodes and high attendee join rates.

.. _Django: https://www.djangoproject.com/


Architecture
::::::::::::

.. image:: _static/b3lb-ref-design.svg

To scale for a huge number of attendees it is possible to:

- scale-out the B3LB API frontends
- scale-out the B3LB polling workers
- scale-out your BBB nodes


Features
::::::::

- multiple b3lb frontend instances
- backend BBB node polling using `Celery <http://celeryproject.org/>`_
- extensive caching based on `Redis <https://redis.io/>`_
- robust against high BBB node response times (i.e. due to ongoing DDoS attacks)


BBB Clustering
--------------

- supports a high number of BBB nodes
- different load balancing factors per cluster
- load calculation by attendees, meetings and CPU load metrics
- maintenance mode allows to disable BBB nodes gracefully


BBB Frontend API
----------------

- deployed on ASGI with `uvicorn <https://www.uvicorn.org/>`_
- HTTP call-outs are implementated async using `aiohttp <https://docs.aiohttp.org/>`_
- support API key rollover using a second secret
- prebuild responses for expensive API calls (``getMeetings``)
- limiting attendees or meetings per tenant
- does not implement but blocks recording API calls


Multitenancy
------------

- per-tenant API hostnames
- start presentation injection
- branding logo injection
- multiple API secrets per tenant


Monitoring
----------

- simple health-check URL
- simple json statistics URL
- prometheus metrics URL


Container Images
::::::::::::::::

B3LB provides in three different docker image provided on `Docker Hub <https://hub.docker.com/search?q=b3lb&type=image>`_ and `GitHub Packages <https://github.com/orgs/DE-IBH/packages?ecosystem=docker>`_. The images can be build from source using the provided `Dockerfiles <https://github.com/DE-IBH/b3lb/tree/main/docker>`_.

.. hint::
    It is intentional that there are no `b3lb:latest` nor `b3lb-static:latest` image tags available. You should always pick a explicit version for your deployment.

b3lb
----

This image contains the Django files of b3lb to run
the ASGI application, Celery tasks and manamgenet CLI commands.

.. tab:: Docker Hub

    ::

        docker pull ibhde/b3lb:1.1.0


.. tab:: GitHub Packages

    ::

        docker pull docker.pkg.github.com/de-ibh/b3lb/b3lb:1.1.0


b3lb-static
-----------

Uses the `Caddy <https://caddyserver.com/>`_ webserver to provide static
assets for the Django admin UI and can be used to publish per-tenant assets.

.. tab:: Docker Hub

    ::

        docker pull ibhde/b3lb-static:1.1.0


.. tab:: GitHub Packages

    ::

        docker pull docker.pkg.github.com/de-ibh/b3lb/b3lb-static:1.1.0



b3lb-dev
--------
This is the development build of b3lb using Djangos single threaded build-in webserver. You should never use this in production.

.. tab:: Docker Hub

    ::

        docker pull ibhde/b3lb-dev:latest


.. tab:: GitHub Packages

    ::

        docker pull docker.pkg.github.com/de-ibh/b3lb/b3lb-dev:latest
