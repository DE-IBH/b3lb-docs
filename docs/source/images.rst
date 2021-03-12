Container Images
================

B3LB provides in three different docker image provided on `Docker Hub <https://hub.docker.com/search?q=b3lb&type=image>`_ and `GitHub Packages <https://github.com/orgs/DE-IBH/packages?ecosystem=docker>`_. The images can be build from source using the provided `Dockerfiles <https://github.com/DE-IBH/b3lb/tree/main/docker>`_.

.. hint::
    It is intentional that there are no `b3lb:latest` nor `b3lb-static:latest` image tags available.

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
