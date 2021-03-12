About
=====

B3LB is designed for large scale-out BBB multitenancy deployments:

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
