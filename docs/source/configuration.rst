Configuration
=============

*b3lb* can be configured using environment variables or a .env file. The following settings are available:

``SECRET_KEY``
    The `secret key <https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key>`_ for your Django deployment.

    **REQUIRED**

.. warning::
    The secret key needs to be unique, secure and kept confidential. A key could be generated using the *pwgen* command::
    
        pwgen -ys 50 1

``DATABASE_URL``
    The `databases setting <https://docs.djangoproject.com/en/3.2/ref/settings/#databases>`_ for your Django deployment. The ``default`` database setting is used and should point to a PostgreSQL database.

    **REQUIRED**

``CELERY_BROKER_URL``
    The `broker URL <https://docs.celeryproject.org/en/stable/userguide/configuration.html#std-setting-broker_url>`_ used by *Celery*. The broker needs to be the same on all *b3lb* instances.

    **REQUIRED**

``CACHE_URL``
    The `caches setting <https://docs.djangoproject.com/en/3.2/ref/settings/#caches>`_ for your Django deployment. The ``default`` cache is used by the worker nodes to cache BBB's ``getMeetings`` XML data. It is recommended to use a redis backend.

    Default: ``locmemcache://b3lb-default-cache``

.. hint::
    It is highly recommended to configure a powerful caching backend like *redis*. Running the worker nodes with limited caching will result in many database read requests with huge result sets. The worker nodes should use a shared cache if network throughput is non-expensive.

``CACHEOPS_REDIS``
    This settings needs to configure a redis backend used for ORM caching using `django-cacheops <https://github.com/Suor/django-cacheops#setup>`_. For frontend nodes it is recommend to use a local redis instance to separate  failure domains.

    Default: ``redis://redis/2``

``CACHEOPS_DEGRADE_ON_FAILURE``
    The ORM caching layer will continue to work with degraded performance if the redis backend is not available.
    
    Default: ``True``

``LANGUAGE_CODE``
    The `language ID <https://docs.djangoproject.com/en/3.2/ref/settings/#language-code>`_ to be used for the admin pages.

    Default: ``en-us``

``TIME_ZONE``
    The `time zone <https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-TIME_ZONE>`_ to be used.

    Default: ``UTC``

``B3LB_API_BASE_DOMAIN``
    The b3lb :ref:`base domain<Prerequisites DNS>`.

    **REQUIRED**

.. hint::
    *redis*  is used in the three different settings ``CACHES``, ``CELERY_BROKER_URL`` and ``CACHEOPS_REDIS``. It is highly recommended to use unique redis database identifiers for each setting.

Create your own ``.env`` file using the following template:

.. hint::
    You might use the templates with Jinja2 and set the variables ``api_base_domain``, ``db_passwd`` and
    ``secret_key`` appropriately.

.. code-block:: ini

    # Uvicorn: number of ASGI workers
    WEB_CONCURRENCY=4

    # Django Basics
    SECRET_KEY={{ secret_key }}
    DEBUG=False

    # Django I18N
    LANGUAGE_CODE=de-de
    TIME_ZONE=Europe/Berlin

    # Django Cache
    CACHE_URL=redis://:{{ redis_secret }}@redis:6379/1

    # Django ORM
    DATABASE_URL=psql://b3lb-user:{{ db_passwd }}@db-host:5432/b3lb-db

    # Django ORM Caching
    CACHEOPS_REDIS=redis://:{{ redis_secret }}@redis:6379/2

    # Configure Celery
    CELERY_BROKER_URL=redis://:{{ redis_secret }}@redis:6379/3

    # B3LB
    B3LB_API_BASE_DOMAIN=api.bbbconf.de
