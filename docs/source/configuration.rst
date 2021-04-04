Configuration
=============

*b3lb* is configured using a Django settings file (``settings.py``). You need to customize the following settings:

``SECRET_KEY``
    The `secret key <https://docs.djangoproject.com/en/3.1/ref/settings/#secret-key>`_ for your Django deployment.

.. warning::
    The secret key needs to be unique, secure and kept confidential. A key could be generated using the *pwgen* command::
    
        pwgen -ys 50 1

``DATABASES``
    The `databases setting <https://docs.djangoproject.com/en/3.1/ref/settings/#databases>`_ for your Django deployment. The ``default`` database setting is used and should point to a PostgreSQL database.

``CELERY_BROKER_URL``
    The `broker URL <https://docs.celeryproject.org/en/stable/userguide/configuration.html#std-setting-broker_url>`_ used by *Celery*. The broker needs to be the same on all *b3lb* instances.

``CACHES``
    The `caches setting <https://docs.djangoproject.com/en/3.1/ref/settings/#caches>`_ for your Django deployment. The ``default`` cache is used by the worker nodes to cache BBB's ``getMeetings`` XML data.

.. hint::
    It is highly recommended to configure a powerful caching backend like *redis*. Running the worker nodes with limited caching will result in many database read requests with huge result sets. The worker nodes should use a shared cache if network throughput is non-expensive.

``CACHEOPS_REDIS``
    This settings needs to configure a redis backend used for ORM caching using `django-cacheops <https://github.com/Suor/django-cacheops#setup>`_. For frontend nodes it is recommend to use a local redis instance to separate  failure domains.

``LANGUAGE_CODE``
    The `language ID <https://docs.djangoproject.com/en/3.1/ref/settings/#language-code>`_ to be used for the admin pages.

``TIME_ZONE``
    The `time zone <https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-TIME_ZONE>`_ to be used.

``API_BASE_DOMAIN``
    The b3lb :ref:`base domain<Prerequisites DNS>`.

``ASSETS_FOLDER_URL``
    The URL prefix to build branding logo URLs.

.. hint::
    *redis*  is used in the three different settings ``CACHES``, ``CELERY_BROKER_URL`` and ``CACHEOPS_REDIS``. It is highly recommended to use unique redis database identifiers for each setting.

Create your own ``settings.py`` using the following template:

.. hint::
    You might use the templates with Jinja2 and set the variables ``api_base_domain``, ``assets_domain``, ``db_passwd`` and
    ``secret_key`` appropriately.

.. code-block:: python

    from loadbalancer.settings_base import *

    # create a secret key
    SECRET_KEY = "{{ secret_key }}"


    # Database
    # https://docs.djangoproject.com/en/3.1/ref/settings/#databases

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'b3lb',
            'USER': 'b3lb',
            'PASSWORD': '{{ db_passwd }}',
            'HOST': 'db',
            'PORT': '5432',
        }
    }

    # Configure Celery
    CELERY_BROKER_URL = 'redis://:{{ redis_secret }}@redis:6379/3"

    # Django Cache
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://:{{ redis_secret }}@redis:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient"
            },
        }
    }

    # Django ORM Caching
    CACHEOPS_REDIS = "redis://:{{ redis_secret }}@redis:6379/2"


    # Internationalization
    # https://docs.djangoproject.com/en/3.1/topics/i18n/

    LANGUAGE_CODE = 'de-de'
    TIME_ZONE = 'Europe/Berlin'


    # B3LB SETTINGS

    API_BASE_DOMAIN = "{{ api_base_domain }}"
    ASSETS_FOLDER_URL = "https://{{ assets_domain }}/logos"
