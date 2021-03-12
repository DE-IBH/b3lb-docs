Configuration
=============

.. warning::
    You need to set secure credentials for all passwords and secrets.

::

    from loadbalancer.settings_base import *

    # create a secret key
    SECRET_KEY = "XXXXXX"


    # Database
    # https://docs.djangoproject.com/en/3.1/ref/settings/#databases

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'b3lb',
            'USER': 'b3lb',
            'PASSWORD': 'XXXXXX',
            'HOST': 'db',
            'PORT': '5432',
        }
    }

    # Configure Celery
    CELERY_BROKER_URL = 'redis://:XXXXXX@redis:6379/3"

    # Expire task results after 1h
    CELERY_RESULT_EXPIRES = 3600

    # Django Cache
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://:XXXXXX@redis:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient"
            },
        }
    }

    # Django ORM Caching
    CACHEOPS_REDIS = "redis://:XXXXXX@redis:6379/2"


    # Internationalization
    # https://docs.djangoproject.com/en/3.1/topics/i18n/

    LANGUAGE_CODE = 'de-de'
    TIME_ZONE = 'Europe/Berlin'


    # B3LB SETTINGS

    API_BASE_DOMAIN = "api.examples.com"
    ASSETS_FOLDER_URL = "https://assets.examples.com/logos"
