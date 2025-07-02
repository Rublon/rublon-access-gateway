# Rublon Access Gateway

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Configuration](#configuration)
4. [Installation](#installation)
5. [Updating the Application](#updating-the-application)
6. [Verification and Troubleshooting](#verification-and-troubleshooting)

<a id="overview"></a>
## Overview

This repository contains a [`compose.yml`](https://github.com/Rublon/rublon-access-gateway/blob/main/compose.yml) file for a production deployment of the **Rublon Access Gateway** on a Linux server.

For comprehensive instructions and advanced configurations, refer to the official documentation: [**https://rublon.com/doc/rag/#linux**](https://rublon.com/doc/rag/#linux).

<a id="prerequisites"></a>
## Prerequisites

Before you start, ensure you have the following:

*   A **Linux server**.
*   **Docker** and **Docker Compose** installed.
*   Valid **SSL certificate** (`cert.pem`) and **private key** (`privkey.pem`) files for your domain.

<a id="configuration"></a>
## Configuration

You must configure the `compose.yml` file before launching the services.

### **1. Set Your Hostname**

In the `compose.yml` file, locate the nginx service and update the `SERVER_NAME` environment variable with your actual domain name.

```yaml
services:
  # ...
  nginx:
    # ...
    environment:
      - SERVER_NAME=rag.yourdomain.com # <-- Change this
    # ...
```

### **2. Configure Paths**

You need to map the host paths to the paths inside the container for configuration data and SSL certificates.

*   **Persistent Data**: The application data is stored in a host directory mapped to `/var/www/rag/mac/config/volume`. By default, this is set to `/srv/rag/volume`. You can change this path if needed. Ensure the directory exists on your host system.
```yaml
services:
  rag:
    # ...
    volumes:
      - /srv/rag/volume:/var/www/rag/mac/config/volume # <-- Change if necessary
    # ...
```

*   **SSL Certificates**: Update the volume paths for `cert.pem` and `privkey.pem` to point to the location of your SSL files on the host machine.
```yaml
services:
  # ...
  nginx:
    # ...
    volumes:
      - /path/to/your/cert.pem:/etc/ssl/certs/rag.pem # <-- Change this
      - /path/to/your/privkey.pem:/etc/ssl/private/rag-key.pem # <-- Change this
    # ...
```

### **3. Networking (Optional)**

By default, the nginx container listens on the following standard ports:

*   **Port 80** for HTTP traffic (redirects to HTTPS).
*   **Port 443** for HTTPS traffic.


These are mapped directly from your host. Ensure these ports are not in use by other applications.

You can change these ports by setting the `NGINX_PORT` and `NGINX_SSL_PORT` environment variables in the `nginx` service and updating the port mappings.

For example, to use port 8080 for HTTP and 8443 for HTTPS:

```yaml
services:
  # ...
  nginx:
    # ...
    environment:
      - SERVER_NAME=rag.yourdomain.com
      - NGINX_PORT=8080
      - NGINX_SSL_PORT=8443
    ports:
      - "8080:8080"
      - "8443:8443"
    # ...
```

<a id="installation"></a>
## Installation

1.  Download the [`compose.yml`](https://github.com/Rublon/rublon-access-gateway/blob/main/compose.yml) file to your server.
2.  Navigate to the directory containing the file.
3.  Run the following command to build and start the containers in detached mode:
```bash
docker compose up -d
```

The application will now be running and accessible at the domain you configured.

<a id="updating-the-application"></a>
## Updating the Application
You can update the Rublon Access Gateway application at any point by pulling the latest container images and restarting the services. To do this, run the following commands:

```bash
docker compose pull
docker compose up -d
```


<a id="verification-and-troubleshooting"></a>
## Verification and Troubleshooting

After starting the services, you can verify that everything is running correctly.

### Checking Container Status

To see if both containers (`rag` and `nginx`) are running, use the following command:

```bash
docker compose ps
```

You should see both services listed with a status of running or up.

### Viewing Logs

If you encounter any issues, viewing the logs is the best way to diagnose the problem.

To view a live stream of logs from all services, run:
```bash
docker compose logs -f
```
To view the logs for a specific service (e.g., `nginx`), use:
```bash
docker compose logs nginx
```
Or for the application itself:
```bash
docker compose logs rag
```