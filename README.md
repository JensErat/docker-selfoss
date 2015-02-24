# selfoss for Docker

(c) 2015 Jens Erat <email@jenserat.de>

Redistribution and modifications are welcome, see the LICENSE file for details.

[selfoss](http://selfoss.aditu.de/) is a new multipurpose rss reader, live stream, mashup, aggregation web application.

This Dockerfile provides a selfoss image.

## Setup

All data is stored in `/var/www/html/data`, which is persisted as a volume, but it might be reasonable to export it to the local file system. Port 80 is exposed for the web application.

### Data Folders

Selfoss requires a bunch of folders in the data directory. Make sure to create them if you export the volume, running following from the data volume's root:

    mkdir cache favicons logs thumbnails sqlite
    chown -R 33 . # UID 33 is www-data inside the container

### Configuration

Usually, the configuration file would be stored in `/var/www/html/config.ini`, and thus not persisted. The image symlinks it into the data directory, so you can edit `/var/www/html/data/config.ini` instead. For configuration directives, follow the [selfoss documentation](http://selfoss.aditu.de/#documentation). Generally, no documentation is required; but strongly recommended (ie., to make it password protected).

### Database

Per default, selfoss relies on SQLite, which should be fully sufficient for a single-user application like selfoss. If you prefer a "real" database management application, the image might be missing PHP modules (patches to the Dockerfile are welcome).

### Nginx Proxy

In case you want to proxy selfoss with Nginx, this is a proposal for the `server { }` directive:

    location ~* ^/(data\/logs|data\/sqlite|config\.ini|\.ht|password) {
        deny all; # Disable temporarilly to generate a password usingi the `/password` page
    }

    location /update {
        proxy_read_timeout 300;
        proxy_pass http://selfoss:80;
    }

    location / {
        proxy_pass http://selfoss:80;
    }


## Running a Container

The most basic run command would be:

    docker run -d \
    	--name 'selfoss' \
    	--publish 80:80 \
    	--volume /srv/selfoss:/var/www/html/data \
    	jenserat/selfoss

## Upgrading and Maintenance

Selfoss will automatically upgrade the database. For other maintenance tasks during upgrades, refer to the [selfoss documentation](http://selfoss.aditu.de/#documentation).

To enter the container and perform maintenance, use (given you named the container `selfoss`):

    docker exec -ti selfoss /bin/bash
