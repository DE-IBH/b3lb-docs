Django Admin Commands
=====================

The `b3lb` container image provides the following additional django-admin commands:

**addnode**
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
    ::

        usage: manage.py checkslides [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                     [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                                     [--skip-checks]

        Check slides in slides folder


**getloadvalues**
    ::

        usage: manage.py getloadvalues [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                               [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                               [--skip-checks]

        Get calculated load values of nodes


**gettenantsecrets**
    ::

        usage: manage.py gettenantsecrets [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                  [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                                  [--skip-checks]

        Get first secret and hostnames of tenants.


**listalltenantsecrets**
    ::

        usage: manage.py listalltenantsecrets [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                                      [--pythonpath PYTHONPATH] [--traceback] [--no-color]
                                      [--force-color] [--skip-checks]

        List all secrets of all tenants


**meetingstats**
    ::

        usage: manage.py meetingstats [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                              [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color]
                              [--skip-checks]

        Get status information of tenant and meetings
