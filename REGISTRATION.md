# The APSIM Registration Website

The apsim registration website is an asp .net core web application running in docker behind a reverse proxy (apache) on a Linux GCP VM (apsim-web). The source code is [on github](https://github.com/APSIMInitiative/APSIM.Registration). The website allows users to register for and download apsim. The website consists of a REST API (RegistrationController) and a single-page application consisting of the landing page, registration form, and download page. The docker configuration is stored in the [apsim-web](https://github.com/hol430/apsim-web) repository. The registration data is stored in a MariaDB database.

The registration site is very closely linked to the builds API. I think in the long run it would make sense to merge/combine the two.

## Registration Emails

The registration emails are sent via [SendGrid](https://app.sendgrid.com/) using a no-reply address on the apsim.info domain (note that this address is not a mailbox). SendGrid relies on some DNS entries (which we manage ourselves).

## Running the Website

The website reads several credentials from environment variables. Some parts of the website will not function correctly if these variables are not set.

- `REGO_DB_CONNECT_STRING`: connection string to the registrations database
- `SMTP_HOST`: Hostname for SMTP service (used to send registration confirmation emails)
- `SMTP_PORT`: Port for SMTP service
- `SMTP_USER`: Username for SMTP service
- `SMTP_TOKEN`: Auth token for SMTP service

In order to run a local debug build, you'll need to run a MariaDB instance. Docker makes this rather easy. A sample docker-compose file might look something like:

```
version: '3.1'

services:
  server:
    image: mariadb:latest
    container_name: registrations-db
    restart: unless-stopped
    environment:
      MARIADB_RANDOM_ROOT_PASSWORD: 1
      MARIADB_USER: ${MARIADB_USER}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD}
      MARIADB_DATABASE: Registrations
    ports:
      - 33060:3306
    volumes:
      - database:/var/lib/mysql
volumes:
  database:
```

And then set the `REGO_DB_CONNECT_STRING` variable appropriately when starting the site: `export REGO_DB_CONNECT_STRING="server=127.0.0.1;port=33060;uid=${MARIADB_USER};pwd=${MARIADB_PASSWORD};database=Registrations"` 

## Deploying Changes

Changes to the website currently must be deployed manually. This can be done by ssh-ing into the production server, cloning or navigating to an existing apsim-web repository, and running the deploy script in the registration directory. This will rebuild the docker image from latest git sources (master branch on the APSIM.Registration repository) and restart the container from the new image. NOTE: before you run the deploy script, ensure that the required environment variables are set correctly; the script will fail without these.

In the long run, I'd like to see this managed by the CI server, as with apsim; have the unit tests run successfully before the PR can be merged and then update the docker container automatically when a pull request is merged.

## Callers/Users

The API is called from the following places:

- APSIM GUI: in the upgrade GUI to register an upgrade with a user account
