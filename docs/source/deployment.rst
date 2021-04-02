Deployment
==========

*b3lb* is distributed as Docker image and the following deployment blueprint is based on `Docker Compose <https://docs.docker.com/compose/>`_. As reverse proxy `traefik <https://github.com/traefik/traefik>`_ is used. The following templates are provided:

``docker-compose.yml``
    The compose file deploying all services on a single node. For scale-out you need some deployment environment allowing you to scale the components.

``conf.d/traefik/config.yml``
    The traefik configuration.

``conf.d/traefik/traefik.yml``
    Static middlewares for traefik used in the compose file.


.. hint::
    You need at least to change the options tagged with ``TODO:`` in the following templates. You might use the templates with Jinja2 and set the variables ``ansible_fqdn``, ``api_base_domain``, ``assets_domain``, ``tsig_key`` and ``tsig_secret`` appropriately.


.. tab:: docker-compose.yml

    .. code-block:: yaml

        version: '3'

        services:
          # reverse proxy
          traefik:
            image: traefik:2.4.7
            read_only: true
            ports:
              - 80:80
              - 443:443
            volumes:
              - ./conf.d/traefik/config.yml:/etc/traefik/config.yml:ro
              - ./conf.d/traefik/traefik.yml:/etc/traefik/traefik.yml:ro
              - ./data.d/traefik/acme:/etc/traefik/acme
              - /var/run/docker.sock:/var/run/docker.sock
            environment:
              RFC2136_DNS_TIMEOUT: 21
              # TODO: nameserver to be used for DNS-01 challenge
              RFC2136_NAMESERVER: 192.0.2.53
              RFC2136_TSIG_ALGORITHM: hmac-sha512.
              # TODO: tsig key used for auth
              RFC2136_TSIG_KEY: {{ tsig_key }}
              RFC2136_TSIG_SECRET: {{ tsig_secret }}
            labels:
              - traefik.enable=true

              # HOST: publish traefik dashboard
              - traefik.http.routers.traefik.entrypoints=https
              - traefik.http.routers.traefik.rule=Host(`{{ ansible_fqdn }}`)
              - traefik.http.routers.traefik.middlewares=management-chain@file
              - traefik.http.routers.traefik.tls=true
              - traefik.http.routers.traefik.tls.options=default
              - traefik.http.routers.traefik.tls.certResolver=acmeTLS
              - traefik.http.routers.traefik.service=api@internal

              # HOST: publish traefik ping service
              - traefik.http.routers.traefik-ping.entrypoints=https
              - traefik.http.routers.traefik-ping.rule=Host(`{{ ansible_fqdn }}`) && PathPrefix(`/ping`)
              - traefik.http.routers.traefik-ping.middlewares=endpoint-chain@file
              - traefik.http.routers.traefik-ping.tls=true
              - traefik.http.routers.traefik-ping.tls.options=default
              - traefik.http.routers.traefik-ping.tls.certResolver=acmeTLS
              - traefik.http.routers.traefik-ping.service=ping@internal

              # HOST: publish traefik prometheus metrics
              - traefik.http.routers.prometheus.entrypoints=https
              - traefik.http.routers.prometheus.rule=Host(`{{ ansible_fqdn }}`) && PathPrefix(`/metrics`)
              - traefik.http.routers.prometheus.middlewares=management-chain@file
              - traefik.http.routers.prometheus.tls=true
              - traefik.http.routers.prometheus.tls.options=default
              - traefik.http.routers.prometheus.tls.certResolver=acmeTLS
              - traefik.http.routers.prometheus.service=prometheus@internal
            networks:
              - rp
            restart: always
            logging: &default_logging
              driver: "json-file"
              options:
                max-size: "1M"
                max-file: "5"

          # b3lb frontend
          django:
            image: ibhde/b3lb:1.2.0
            volumes:
              - ./data.d/b3lb/logos:/usr/src/app/rest/logos:ro
              - ./data.d/b3lb/slides:/usr/src/app/rest/slides:ro
              - ./conf.d/b3lb/settings.py:/usr/src/app/loadbalancer/settings.py:ro
            environment:
              # TODO: Adjust number of uvicorn processes!
              WEB_CONCURRENCY: 4
            labels:
              # TENANT: /bigbluebutton/api
              - traefik.enable=true
              - traefik.http.routers.api.entrypoints=https
              - traefik.http.routers.api.rule=HostRegexp(`{tenant:[a-z0-9-]+}.{{ api_base_domain }}`) && PathPrefix(`/bigbluebutton/api/`)
              - traefik.http.routers.api.middlewares=endpoint-chain@file
              - traefik.http.routers.api.tls=true
              - traefik.http.routers.api.tls.options=default
              - traefik.http.routers.api.tls.certResolver=acmeDNS
              - "traefik.http.routers.api.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.api.tls.domains[0].sans=*.{{ api_base_domain }}"
              - traefik.http.routers.api.service=api
              - traefik.http.services.api.loadbalancer.server.port=8000

              # GLOBAL: /b3lb/ping
              # TENANT: /b3lb/ping
              - traefik.http.routers.api-ping.entrypoints=https
              - traefik.http.routers.api-ping.rule=(HostRegexp(`{{ api_base_domain }}`) || HostRegexp(`{tenant:[a-z0-9-]+}.{{ api_base_domain }}`)) && Path(`/b3lb/ping`)
              - traefik.http.routers.api-ping.middlewares=endpoint-chain@file
              - traefik.http.routers.api-ping.tls=true
              - traefik.http.routers.api-ping.tls.options=default
              - traefik.http.routers.api-ping.tls.certResolver=acmeDNS
              - "traefik.http.routers.api-ping.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.api-ping.tls.domains[0].sans=*.{{ api_base_domain }}"
              - traefik.http.routers.api-ping.service=api-ping
              - traefik.http.services.api-ping.loadbalancer.server.port=8000

              # TENANT: /b3lb/stats
              # TENANT: /b3lb/metrics
              - traefik.http.routers.stats.entrypoints=https
              - traefik.http.routers.stats.rule=HostRegexp(`{tenant:[a-z0-9-]+}.{{ api_base_domain }}`) && (Path(`/b3lb/stats`) || Path(`/b3lb/metrics`))
              - traefik.http.routers.stats.middlewares=endpoint-chain@file
              - traefik.http.routers.stats.tls=true
              - traefik.http.routers.stats.tls.options=default
              - traefik.http.routers.stats.tls.certResolver=acmeDNS
              - traefik.http.routers.stats.service=stats
              - traefik.http.services.stats.loadbalancer.server.port=8000
              - "traefik.http.routers.stats.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.stats.tls.domains[0].sans=*.{{ api_base_domain }}"

              # GLOBAL: /b3lb/metrics
              - traefik.http.routers.b3lb-metrics.entrypoints=https
              - traefik.http.routers.b3lb-metrics.rule=Host(`{{ api_base_domain }}`) && Path(`/b3lb/metrics`)
              - traefik.http.routers.b3lb-metrics.middlewares=management-chain@file
              - traefik.http.routers.b3lb-metrics.tls=true
              - traefik.http.routers.b3lb-metrics.tls.options=default
              - traefik.http.routers.b3lb-metrics.tls.certResolver=acmeDNS
              - traefik.http.routers.b3lb-metrics.service=b3lb-metrics
              - traefik.http.services.b3lb-metrics.loadbalancer.server.port=8000
              - "traefik.http.routers.b3lb-metrics.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.b3lb-metrics.tls.domains[0].sans=*.{{ api_base_domain }}"

              # GLOBAL: /admin/
              - traefik.http.routers.admin.entrypoints=https
              - traefik.http.routers.admin.rule=Host(`{{ api_base_domain }}`) && (PathPrefix(`/admin/`) || Path(`/b3lb/metrics`))
              - traefik.http.routers.admin.middlewares=management-chain@file
              - traefik.http.routers.admin.tls=true
              - traefik.http.routers.admin.tls.options=default
              - traefik.http.routers.admin.tls.certResolver=acmeDNS
              - traefik.http.routers.admin.service=admin
              - traefik.http.services.admin.loadbalancer.server.port=8000
              - "traefik.http.routers.admin.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.admin.tls.domains[0].sans=*.{{ api_base_domain }}"
            networks:
              - rp
              - lb
            restart: always
            logging:
              <<: *default_logging

          # static assets: logos, slides and Django admin
          static:
            image: ibhde/b3lb-static:1.2.0
            labels:
              # Django admin static assets
              - traefik.enable=true
              - traefik.http.routers.static.entrypoints=https
              - traefik.http.routers.static.rule=Host(`{{ api_base_domain }}`) && PathPrefix(`/static/`)
              - traefik.http.routers.static.middlewares=management-chain@file,static-strip
              - traefik.http.routers.static.tls=true
              - traefik.http.routers.static.tls.options=default
              - traefik.http.routers.static.tls.certResolver=acmeDNS
              - traefik.http.middlewares.static-strip.stripprefix.prefixes=/static
              - traefik.http.services.static.loadbalancer.server.port=8001
              - "traefik.http.routers.static.tls.domains[0].main={{ api_base_domain }}"
              - "traefik.http.routers.static.tls.domains[0].sans=*.{{ api_base_domain }}"

              # logo & slide assets
              - traefik.enable=true
              - traefik.http.routers.assets.entrypoints=https
              - traefik.http.routers.assets.rule=Host(`{{ assets_domain }}`)
              - traefik.http.routers.assets.middlewares=endpoint-chain@file
              - traefik.http.routers.assets.tls=true
              - traefik.http.routers.assets.tls.options=default
              - traefik.http.routers.assets.tls.certResolver=acmeDNS
              - traefik.http.services.assets.loadbalancer.server.port=80
              - "traefik.http.routers.assets.tls.domains[0].main={{ assets_domain }}"
            volumes:
              - ./data.d/b3lb/logos:/usr/share/caddy/logos:ro
              - ./data.d/b3lb/slides:/usr/share/caddy/slides:ro
            networks:
              - rp
            restart: always
            logging:
              <<: *default_logging

          # celery scheduling
          celery-beat:
            image: ibhde/b3lb:1.2.0
            command: celery-beat
            volumes:
              - ./conf.d/b3lb/settings.py:/usr/src/app/loadbalancer/settings.py:ro
            networks:
              - lb
            restart: always
            logging:
              <<: *default_logging

          # celery worker
          celery-tasks:
            image: ibhde/b3lb:1.2.0
            command: celery-tasks
            volumes:
              - ./data.d/b3lb/slides:/usr/src/app/rest/slides:ro
              - ./conf.d/b3lb/settings.py:/usr/src/app/loadbalancer/settings.py:ro
            networks:
              - lb
            restart: always
            logging:
              <<: *default_logging

          # cache
          redis:
            image: redis:6.0.12-alpine
            # TODO: Adjust max memory!
            # TODO: Set your redis secret!
            command: redis-server --maxmemory 4096mb --maxmemory-policy volatile-lfu --requirepass {{ redis_secret }}
            networks:
              - lb
            restart: always
            logging:
              <<: *default_logging

        networks:
          # b3lb internal
          lb:

          # reverse proxy
          rp:

.. tab:: conf.d/traefik/config.yml

    .. code-block:: yaml

        http:
          middlewares:
            # add security related http headers
            # https://doc.traefik.io/traefik/middlewares/headers/#configuration-options
            security-headers:
              headers:
                frameDeny: true
                sslRedirect: true
                browserXssFilter: true
                contentTypeNosniff: true
                forceSTSHeader: true
                stsSeconds: 31536000
                stsIncludeSubdomains: true
                stsPreload: true

            # prevent search enginge indexing
            x-robots-tag:
               headers:
                  customResponseHeaders:
                     X-Robots-Tag: "noindex, nofollow, noarchive, nosnippet, notranslate, noimageindex"

            # list of ip prefixes allowed to access management and metrics 
            mgmt-whitelist:
              ipWhiteList:
                sourceRange:
                  # TODO: Add your management ip prefixes!
                  - 127.0.0.0/8


            # middleware chain used for public endpoints
            endpoint-chain:
              chain:
                middlewares:
                - security-headers
                - x-robots-tag

            # middleware chain used for management endpoints
            management-chain:
              chain:
                middlewares:
                - security-headers
                - x-robots-tag
                - mgmt-whitelist


        tls:
          options:
            # TLS settings
            default:
              sniStrict: true
              minVersion: VersionTLS12
              preferServerCipherSuites: true
              cipherSuites:
                - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
                - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
                - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
                - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
                - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
                - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
                - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
              curvePreferences:
                - X25519
                - CurveP521
                - CurveP384

.. tab:: conf.d/traefik/traefik.yml

    .. code-block:: yaml

        # disable traefik call home
        global:
          checkNewVersion: false
          sendAnonymousUsage: false

        # enable traefik dashboard
        api:
          dashboard: true

        # enable traefik ping handler
        ping:
          manualRouting: true

        # enable traefik prometheus metrics export
        metrics:
          prometheus:
            manualRouting: true


        # entrypoints for http and https
        entryPoints:
          http:
            address: ":80"
            http:
              redirections:
                entryPoint:
                  to: https
                  scheme: https
          https:
            address: ":443"
            http:
              tls:
                options: default

        # add docker and file providers
        providers:
          docker:
            endpoint: "unix:///var/run/docker.sock"
            watch: true
            exposedByDefault: false
            # Needs to match the network name created by
            # docker-compose!
            network: b3lb_rp
          file:
            filename: /etc/traefik/config.yml

        # frontend certificates
        certificatesResolvers:
          acmeDNS:
            acme:
              # TODO: Adding an email address is required!
              #email: 
              storage: /etc/traefik/acme/acmeDNS.json
              dnsChallenge:
                provider: rfc2136

        # use default logging
        log: {}

        # enable access logging only for failed or high latency requests
        accessLog:
          filters:
            statusCodes:
              - "400-499"
              - "500-599"
            retryAttempts: true
            minDuration: "500ms"
