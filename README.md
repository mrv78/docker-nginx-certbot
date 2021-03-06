# docker-nginx-certbot
nginx in docker doing SSL termination with certbot

This nginx instance forwards all requests to upstreams on the same docker networks.

Connect your HTTP containers to the `<prefix>_default` network, map the alias for the given
`Host` header to a container and you are good to go.

# Requirements
- docker >= 1.19
- docker-compose >= 1.8

# Installation
- clone this repo
- edit `.env` and set your `EMAIL` and `DOMAINS` variables

The `DOMAINS` variable is expanded using bash, any whitespace is replaced with commas. This way you can define a bunch of subdomains at once.

# certbot terms of service

`certbot` is a frontend to handle [letsencrypt](https://letsencrypt.org/) SSL certificates.
This image uses these services and you must accept their terms of service.

Right now, I cannot find an URL to point you there, but be sure to check them out.

When installing the initial certificate, certbot will ask you to accept the TOS (so run it interactive). Or you can set the `TOS=--agree-tos` environment variable in `.env`.

# Rate limits

Be sure to read https://certbot.eff.org/faq/#what-are-the-current-rate-limits
before starting experiments with this image.

# Get your initial certificate

Run `certbot.sh` in the docker container. It'll see that it didn't fetch any certifcates yet and run the inital setup.

    docker-compose exec nginx /etc/nginx/ssl/certbot.sh -v
    
If you haven't set the `TOS` environment variable `certbot` will ask you to accept their TOS, so be sure to run this command from an interactive shell.

# Update your certificate

`certbot` certificates are only valid for some 90 days. You'll need to refresh them regularly.
The procedure is the same as for the inital step.

Run `certbot.sh` in the docker container. It'll see the existing certificate and run the updater.

    docker-compose exec nginx /etc/nginx/ssl/certbot.sh -v
    
Put this line in your docker-host crontab and run it every month or so.

# Keeping and sharing your certificate

SSL related files are placed in `/etc/letsencrypt/live/<firstdomain>/`. This
directory is recreated with the container, unless you choose to make a volume
and keep this volume.

The `docker-compose.yml` is configured to mount a local `./letsencrypt` directory that
will be populated with all the certbot data.

Edit the `volumes` section `docker-compose.yml` to adapt this to your needs.
