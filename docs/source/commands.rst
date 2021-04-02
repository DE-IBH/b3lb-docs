Django Admin Commands
=====================

The `b3lb` container image provides the following additional django-admin commands:

**addnode**
    Creates a BBB node and adds it to a existing cluster.

    ::

        usage: manage.py addnode [-h] --slug SLUG --secret SECRET --cluster CLUSTER [--version] [-v {0,1,2,3}]
                         [--settings SETTINGS] [--pythonpath PYTHONPATH] [--traceback] [--no-color]
                         [--force-color] [--skip-checks]

        Add new BBB cluster node.

        optional arguments:
          --slug SLUG           hostname
          --secret SECRET       BBB API secret
          --cluster CLUSTER     cluster name


**addsecrets**
    Generates tenant API sub secrets using random credentials.

    ::

        usage: manage.py addsecrets [-h] --tenant-slug TENANT_SLUG --sub-id SUB_ID [--version] [-v {0,1,2,3}]
                                    [--settings SETTINGS] [--pythonpath PYTHONPATH] [--traceback] [--no-color]
                                    [--force-color] [--skip-checks]

        Add new tenant secret(s).

        optional arguments:
          --tenant-slug TENANT_SLUG
                                Slug
          --sub-id SUB_ID       Ids from N-M or single id, with 0 <= N, M < 1000


**checkslides**
    Synchronizes the slides objects to the existing slide files.

    ::

        usage: manage.py checkslides [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                     [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                                     [--skip-checks]

        Check slides in slides folder


**getloadvalues**
    Dumps the BBB node's load values.

    ::

        usage: manage.py getloadvalues [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                               [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                               [--skip-checks]

        Get calculated load values of nodes


**gettenantsecrets**
    Dumps all API secrets of a single tenant.

    ::

        usage: manage.py gettenantsecrets [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                  [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                                  [--skip-checks]

        Get first secret and hostnames of tenants.


**listalltenantsecrets**
    Dumps API secrets of all tenants.

    ::

        usage: manage.py listalltenantsecrets [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                      [--pythonpath PYTHONPATH] [--traceback] [--no-color]
                                      [--force-color] [--skip-checks]

        List all secrets of all tenants


**meetingstats**
    Dump statistics.

    ::

        usage: manage.py meetingstats [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                              [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                              [--skip-checks]

        Get status information of tenant and meetings
