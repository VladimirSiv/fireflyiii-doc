# How to install Firefly III using Docker

## Docker Compose

### Download Docker Compose file

Download [the Docker Compose file](https://raw.githubusercontent.com/firefly-iii/docker/main/docker-compose.yml) and place it somewhere convenient. Use a dedicated directory.  To include the Data Importer in your installation, please read [how to install the data importer](../../data-importer/installation/docker.md).

Grab the raw file, and don't copy-paste the text from your browser. The spaces in the file are very important. So use "Save As".

!!! info
    The Apache server inside this Docker image will run as `www-data`. This will be reflected by the files you upload: they will be owned by `www-data`. You can change the user the image runs under but that user must exist inside the Docker image or things may not work as expected.

### Download environment variables

There are two environment variable-files you need to run this Docker Compose file. Download all files and save them next to the Docker Compose file.

- The first file contains Firefly III variables and can be downloaded from [the Firefly III repository](https://raw.githubusercontent.com/firefly-iii/firefly-iii/main/.env.example). Save it as a new file called `.env`.
- The second file contains the database variables and can be downloaded from [the Docker repository](https://raw.githubusercontent.com/firefly-iii/docker/main/database.env). Save it as a new file called `.db.env`.

It is **important** that you rename the file as instructed here. You can see in the Docker compose file why this is. There is a reference to it: `env_file:`. If you don't name it as it is in the Docker Compose file, you must edit the Docker compose file to match the file names.

If you include the data importer, you MUST do this:

1. Change `FIREFLY_III_URL` in `.importer.env` to `http://app:8080`

Either way, you should also do this (not mandatory):

1. Change `DB_PASSWORD` in `.env` to something else. Pick a nice password.
2. Change `MYSQL_PASSWORD` in `.db.env` to the SAME value

!!! note
    Change the password FIRST. If you change the password *after* you started Docker, it will complain about having no access.

### Start the container

Run the following command in the directory where both `docker-compose.yml` and all environment variable files are present.

```bash
docker compose -f docker-compose.yml up -d --pull=always
```

You can follow the progress of the installation by running this command:

```text
docker compose -f docker-compose.yml logs -f
```

When the installation is done, Firefly III will thank you for installing it. Once you see this message, you can visit Firefly III. It will be running at your [localhost](http://localhost).

### Surf to Firefly III

You can now visit Firefly III at [http://localhost](http://localhost) or [http://docker-ip:port](http://docker-ip:port) if it is running on a custom port. To continue, read [the tutorial on how to create accounts and transactions](../../../tutorials/finances/first-steps.md).

## Docker Hub

The instructions in this section will help you set up a single container.

With these commands you create one container: the container for Firefly III itself. If you do this, you should already have a MySQL or a Postgres database running somewhere. Without such a database container, Firefly III will **not** work.

### Create a volume

This volume is used to persistently store uploaded files.

```text
docker volume create firefly_iii_upload
```

### Start the container

Run this Docker command to start the Firefly III container. Edit the environment variables to match your own database. You should really change the `APP_KEY` as well. It should be a random string of _exactly_ 32 characters. You can generate such a key with the following command: 

```bash
head /dev/urandom | LC_ALL=C tr -dc 'A-Za-z0-9' | head -c 32 && echo
```

```text
docker run -d \
-v firefly_iii_upload:/var/www/html/storage/upload \
-p 80:8080 \
-e APP_KEY=CHANGEME_32_CHARS \
-e DB_HOST=CHANGEME \
-e DB_PORT=3306 \
-e DB_CONNECTION=mysql \
-e DB_DATABASE=CHANGEME \
-e DB_USERNAME=CHANGEME \
-e DB_PASSWORD=CHANGEME \
fireflyiii/core:latest
```

Firefly III assumes that you're using MySQL. If you use PostgreSQL, change the following environment variable in the command: `DB_CONNECTION=pgsql` and change the port, `DB_PORT=5432`.

When executed this command will fire up a Docker container with Firefly III inside of it. It may take some time to start. If the database is set up properly it will automatically migrate and install a default database, and you should be able to surf to your container (usually located at [localhost](http://localhost)) to use Firefly III.

!!! info
    The Apache server inside this Docker image will run as `www-data`. This will be reflected by the files you upload: they will be owned by `www-data`. You can change the user the image runs under but that user must exist inside the Docker image or things may not work as expected.

## Docker tags

The instructions always assume `fireflyiii/core:latest`. This is the latest stable release. Other tags are:

* `fireflyiii/core:beta`. This tag contains beta releases.
* `fireflyiii/core:alpha`. This tag contains alpha releases.
* `fireflyiii/core:develop`. Always the latest develop image. Maybe unstable.

All Docker tags are built for ARMv7, ARM64 and AMD64. ARMv6 is not included, so these images will *not* work on the Raspberry Pi Zero, Raspberry Pi 1 (A+B) or Raspberry Pi Compute Module.

## Docker and reverse proxies

In the `.env` file you downloaded you will find a variable called `TRUSTED_PROXIES` which must be set to either the reverse proxy machine or simply `*`. Set `APP_URL` to the URL Firefly III will be on. For example:

```text
# ...
APP_URL=https://firefly.example.com
TRUSTED_PROXIES=*
# ...
```

On the command line, this would be something like:

```text
-e DB_HOST=fireflyiiidb \
-e DB_DATABASE=firefly \
-e DB_USERNAME=firefly \
-e DB_PORT=3306 \
-e DB_CONNECTION=mysql \
-e DB_PASSWORD=somepw \
-e APP_KEY=CHANGEME_32_CHARS \
-e APP_URL=https://firefly.example.com \
-e TRUSTED_PROXIES=*
```

If you want to enable add SSL as well, check out [the FAQ](../../../references/faq/install.md).

## Supported Docker environment variables

There are many environment variables that you can set in Firefly III. Just check out the [default env file](https://raw.githubusercontent.com/firefly-iii/firefly-iii/main/.env.example) that lists them all.
