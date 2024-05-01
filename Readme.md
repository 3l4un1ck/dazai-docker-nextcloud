# Deploying Nextcloud with Docker and Configuring Nginx for Reverse Proxy

In this comprehensive guide, we will walk you through the process of deploying Nextcloud using Docker and configuring Nginx as a reverse proxy. Nextcloud is a powerful open-source file synchronization and sharing solution that allows you to store, access, and share your files securely. Docker provides a convenient way to package applications and their dependencies into containers, making deployment easier and more efficient. Nginx will serve as a reverse proxy to handle incoming requests and route them to the appropriate Nextcloud container.

## Prerequisites

Before we begin, ensure that you have Docker and Docker Compose installed on your system. Additionally, you will need a domain name pointed to your server's IP address for Nginx configuration.

## Step 1: Setting Up the Directory Structure

Start by creating a directory for your Nextcloud installation. Navigate to a suitable location on your server and create a new directory:

```bash
mkdir nextcloud
cd nextcloud
```

Inside the `nextcloud` directory, create a `docker-compose.yml` file with the following content:

```yaml
version: '3'

services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    restart: always
    volumes:
      - ./db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: HDf9Z9ZGahBXFoj449KeM3uuebM2UL
      MYSQL_PASSWORD: Qm43D2w3S9TQpVcFGRcsXH8jKjL5Vf
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud

  app:
    image: nextcloud
    ports:
      - 8080:80
    links:
      - db
    volumes:
      - ./nextcloud:/var/www/html
    restart: always
    environment:
      MYSQL_HOST: db
      MYSQL_PASSWORD: Qm43D2w3S9TQpVcFGRcsXH8jKjL5Vf
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud

volumes:
  db:
```

This Docker Compose file defines two services: `db` for MariaDB (MySQL) and `app` for the Nextcloud application. It also specifies volumes for persistent storage.

## Step 2: Deploying Nextcloud with Docker Compose

Now, let's deploy Nextcloud using Docker Compose. Run the following command in the terminal:

```bash
docker-compose up -d
```

This command will pull the necessary Docker images and start the Nextcloud containers in the background. It may take a few moments to complete.

## Step 3: Configuring Nginx as a Reverse Proxy

Next, we need to configure Nginx to act as a reverse proxy for the Nextcloud application. Create a new Nginx configuration file for Nextcloud:

```bash
sudo nano /etc/nginx/sites-available/nextcloud
```

Paste the following configuration into the file, replacing `your_domain.com` with your actual domain name:

```nginx
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Add any additional Nginx configuration directives here
}

```

Save and close the file. Then, create a symbolic link to enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
```

Test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```

If the syntax is okay, reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

## Step 4: Accessing Nextcloud

You can now access Nextcloud through your domain name. Open a web browser and navigate to `http://your_domain.com`. Follow the on-screen instructions to complete the Nextcloud setup.

## Conclusion

In this guide, we have deployed Nextcloud using Docker and configured Nginx as a reverse proxy to access it securely over the internet. By following these steps, you can set up your own Nextcloud instance for file storage and sharing with ease. Make sure to regularly update both Nextcloud and Nginx to maintain security and stability.