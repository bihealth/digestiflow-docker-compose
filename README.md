# Run Digestiflow Server using Docker Compose

Detailed documentation can be found in the "Installation" section of the [Digestiflow Server Manual](https://digestiflow-server.readthedocs.io/en/latest/).
You can get started in four (actually three) easy steps (given that you fulfill the requirements from below).

> - :interrobang: Questions? Need Help?
>   Digestiflow is academic software but we are happy to provide support on a best-effort manner.
>   Use the [issue tracker](https://github.com/bihealth/digestiflow-docker-compose/issues) or send an email to cubi-helpdesk@bihealth.de in case of any problems.
> - :factory: Bringing Digestiflow into "production"?
>   Feel free to contact us via cubi-helpdesk@bihealth.de in the case that you want to use Digestiflow beyond an evaluation.
>   We will try to assist you in your setup on a best-effort manner.

## Prerequisites

- Hardware:
    - Memory: 16 GB of RAM
    - CPU: 2 cores
    - Disk: 10 GB of free disk space
- Operating System:
    - a modern Linux that is [supported by Docker](https://docs.docker.com/engine/install/#server)
    - outgoing HTTPS connections to the internet are allowed to download data and Docker images
    - server ports 80 and 443 are open and free on the host that run on this on
- Software:
    - [Docker](https://docs.docker.com/get-docker/)
    - [Docker Compose](https://docs.docker.com/compose/install/)

## Quickstart

See the "Installation" section of the [Digestiflow Server Manual](https://digestiflow-server.readthedocs.io/en/latest/).

Optionally, fork this repository as a first step so you can track changes that you make using Git.

### 1. Get Scripts and Configuration

Clone this repository (or your clone) with the Docker Compose file.

```bash
$ git clone https://github.com/bihealth/digestiflow-docker-compose.git
$ cd digestiflow-docker-compose
```

### 2. Initialize Volumes and Adjust Configuration

Use the provided `init.sh` script for creating the required volumes (directories).

```bash
$ bash init.sh
$ ls volumes/
postgres  irods  redis  traefik
```

Then copy the example configuration to `.env` and adjust it.

```bash
$ cp env.example .env
$ $EDIT .env
```

### 3. Bring up the site

You can now bring up the site with Docker Compose.
The site will come up at your server and listen on ports 80 and 443 (make sure that the ports are open), you can access it at `https://<your-host>/` in your web browser.
This will create a lot of output and will not return you to your shell.
You can stop the servers with `Ctrl-C`.

```bash
$ docker-compose up
```

You can also use let Docker Compose run the containers in the background:

```bash
$ docker-compose up -d
## XXX TODO XXX ##
```

You can check that everything is running:

```bash
$ docker-compose ps
                          Name                                        Command               State                    Ports                  
--------------------------------------------------------------------------------------------------------------------------------------------
digestiflow-docker-compose_digestiflow-celerybeat_1        docker-entrypoint.sh celer ...   Up      8080/tcp                                
digestiflow-docker-compose_digestiflow-celeryd-default_1   docker-entrypoint.sh celeryd     Up      8080/tcp                                
digestiflow-docker-compose_digestiflow-web_1               docker-entrypoint.sh wsgi        Up      8080/tcp                                
digestiflow-docker-compose_postgres_1                      docker-entrypoint.sh postgres    Up      5432/tcp                                
digestiflow-docker-compose_redis_1                         docker-entrypoint.sh redis ...   Up      6379/tcp                                
digestiflow-docker-compose_traefik_1                       /entrypoint.sh --providers ...   Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
```

In the case of any error please report it to us via the Issue Tracker of this repository or email to `cubi-helpdesk@bihealth.de`.
Please include the full output as a text file attachment.

### 4. Create Superuser

Create your first super user (call it `root` or adjust `.env`).

```bash
$ docker-compose exec digestiflow-web python /usr/src/app/manage.py createsuperuser \
  --username root
Email address: 
Password: 
Password (again): 
Superuser created successfully.
```

### 5. Use Digestiflow

Visit the website at `https://<your-host>/` and login with the account `root` and password that you just entered.
There will be a warning about self-signed certificates, see [TLS/SSL Certificates](#tlsssl-certificates) below on how to deal with this.
You can change it in the `Django Admin` (available from the menu with the little user icon on the top right).
You can also use the Django Administration interface to create new user records.

## Anatomy of this Repository

- The file `init.sh` performs some initialization to be done before starting the containers.
- The file `docker-compose.yml` contains the definition of the services required to run Digestiflow Server.
- The `config` directory contains files that are used for configuration:
    - `config/traefik/certificates.toml` -- configuration for SSL/TLS setup
        - `/etc/traefik/tls/server.crt` and `server.key` -- place the SSL certificate and unencrypted private key here for custom SSL certificates
        - in the case of certificates from DFN (used by many German academic organizations), you will have to provide the full chain to the "Telekom" certificate in `server.crt`.

```bash
$ tree
.
├── config
│   ├── exomiser
│   │   └── application.properties
│   ├── postgres
│   │   └── postgresql.conf.orig
│   └── traefik
│       ├── certificates.toml
│       └── ssl
│           └── PLACE_TLS_FILES_HERE
├── docker-compose.yml
├── LICENSE
├── README.md
└── init.sh
```

### TLS/SSL Certificates

The setup uses [traefik](https://traefik.io/) as a reverse proxy and must be reconfigured if you want to change the default behaviour of using self-signed certificates.

- `settings:testing` --
  By default (and as a fallback), traefik will use self-signed certificates that are recreated at every startup.
  These are probably fine for a test environment but you might want to change this to one of the below.
- `settings:production-provide-certificate` --
  You can provide your own certificates by placing them into ``config/traefik/tls/server.crt` and `server.key`.
  Make sure to provide the full certificate chain if needed (e.g., for DFN issued certificates).
- `settings:production-letsencrypt` --
  If your site is reachable from the internet then you can also use `settings:production-letsencrypt` which will use [letsencrypt](https://letsencrypt.org/) to obtain the certificates.
  NB: if you make your site reachable from the internet then you should be aware of the implications.
  Digestiflow is MIT licensed software which means that it comes "without any warranty of any kind", see the `LICENSE` file for details.

After changing the configuration, restart the site (e.g., with `docker-compose down && docker-compose up -d` if it is runnin in detached mode).


## Maintainer Info

This section section is only interesting for maintainers of `digestiflow-docker-compose`.

Install the Github CLI ([see instructions](https://github.com/cli/cli#installation)), then login with `gh auth login`.

### Creating a new Release

Use `${digestiflow-server-version}-${build-version}` as the tag name for `digestiflow-docker-compose`.
This allows people to easily track if something changed here but the `digestiflow-server` version is the same.

1. Create a new entry in `HISTORY.md` and commit.
2. Create a new tag: `make tag TAG=vxx`.
3. Push the tag and release: `make release`.
