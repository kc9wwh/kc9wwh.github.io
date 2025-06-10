---
title: "Getting started with Fleet"
description: "Quick and easy setup with Docker!"
date: 2025-06-09T12:33:38Z
image: cover.jpg
categories: [Tech]
tags: [Fleet, Docker, Open Source, MySQL, Redis]
hidden: false
comments: true
draft: true
---
# Introduction: Unleash the Power of Fleet with Docker!
In today's complex digital landscape, achieving comprehensive endpoint visibility is paramount for robust security and efficient IT operations. Fleet, an open-source platform, leverages the power of osquery to provide deep insights into device fleets, covering Linux, macOS, Windows, and even Chromebooks. It's a versatile tool for vulnerability reporting, detection engineering, device management (MDM), and monitoring device health. However, setting up any powerful platform can sometimes seem daunting, particularly when considering resource allocation for testing or initial exploration.

This is where the magic of Docker comes into play. This guide demonstrates how to launch a fully functional Fleet environment—complete with its necessary database and caching layer—using individual `docker run` commands. This approach offers a lightweight, fast, and isolated method to get hands-on with Fleet, making it ideal for IT and security professionals, DevOps engineers, or anyone curious to explore osquery management at scale with minimal upfront commitment.

## Why Docker for Fleet? The "Easy Button" Approach
Utilizing Docker for deploying Fleet presents several compelling advantages, especially for evaluation, development, or smaller-scale use cases:

- **Simplicity**: Docker encapsulates each component of the Fleet stack (Fleet server, MySQL database, Redis cache) into its own container. This packaging simplifies deployment and dependency management.
- **Speed**: With a few Docker commands, the entire stack can be provisioned and running in minutes. Similarly, tearing down the environment is just as quick, leaving the host system clean.
- **Isolation**: Containers run in isolated environments, preventing conflicts with other software or configurations on the host machine. This is perfect for testing without impacting existing setups.
- **Resource Efficiency**: Docker containers are generally more lightweight than full virtual machines. This makes it feasible to run Fleet on a local machine or a small server for testing purposes.
- **Consistency**: Docker ensures that the Fleet environment is consistent across different machines, from a developer's laptop to a test server.

## Prerequisites: What's Needed
To embark on this Dockerized Fleet journey, only a few items are necessary:

- Docker Engine installed on the host system.
- Basic familiarity with command-line interface (CLI) operations.
- A spirit of exploration to dive into the capabilities of Fleet!

## The Architecture: The Dockerized Fleet Stack
The setup will consist of the following components, all running in separate Docker containers:

1.  `fleetdm/fleet:latest`: The core Fleet server application.
2.  `mysql:8.0.36`: A MySQL database instance to store Fleet's metadata.
3.  `redis`: A Redis instance used by Fleet for caching and managing live query sessions.

These containers will communicate over the host network, so it's important to ensure the IP address used for configuration is accessible from the containers.

## Let's Get Building! Step-by-Step with Docker Run
This section walks through the commands needed to bring the Fleet stack to life.

### Step 1: Prepare Host Directories
Before launching the containers, you need to create the directories on your host machine that will be used for persistent storage. This ensures your data isn't lost when containers are stopped or removed.

```bash
# Create directories for MySQL
mkdir -p /mnt/user/appdata/mysql/{data,logs,conf.d,initdb}

# Create directory for Fleet
mkdir -p /mnt/user/appdata/fleetdm
```

### Step 2: Launch the MySQL Container
Run the following command to start the MySQL database. This command maps ports and volumes for data persistence and logging.

```bash
docker run \
  -d \
  --name='MySQL' \
  --net='bridge' \
  --pids-limit 2048 \
  --privileged=true \
  -e TZ="America/Chicago" \
  -e 'MYSQL_DATABASE'='fleet' \
  -e 'MYSQL_USER'='fleet-user' \
  -e 'MYSQL_PASSWORD'='fleet-user-password' \
  -e 'MYSQL_ROOT_PASSWORD'='super-secret-password' \
  -p '3306:3306/tcp' \
  -v '/mnt/user/appdata/mysql/data':'/var/lib/mysql':'rw' \
  -v '/mnt/user/appdata/mysql/logs':'/var/log/mysql':'rw' \
  -v '/mnt/user/appdata/mysql/conf.d':'/etc/mysql/conf.d':'rw' \
  -v '/mnt/user/appdata/mysql/initdb':'/docker-entrypoint-initdb.d':'rw' \
  --memory=2G \
  --restart=unless-stopped \
  'mysql:8.0.36' mysqld \
  --log-error=/var/log/mysql/error.log
```

### Step 3: Launch the Redis Container
Next, start the Redis container. It's a simpler setup, primarily needing a port mapping.

```bash
docker run \
  -d \
  --name='Redis' \
  --net='bridge' \
  --pids-limit 2048 \
  -e TZ="America/Chicago" \
  -p '6379:6379/tcp' \
  'redis'
```

### Step 4: Launch the Fleet Server
Finally, launch the Fleet server. This command is the most extensive, as it includes configuration details to connect to the MySQL and Redis containers.

**Note:** Make sure to replace `192.168.50.129` in the `FLEET_MYSQL_ADDRESS` and `FLEET_REDIS_ADDRESS` variables with the actual IP address of your Docker host.

```bash
docker run \
  -d \
  --name='fleetdm' \
  --net='bridge' \
  --pids-limit 2048 \
  -e TZ="America/Chicago" \
  -e 'FLEET_MYSQL_ADDRESS'='192.168.50.129:3306' \
  -e 'FLEET_MYSQL_DATABASE'='fleet' \
  -e 'FLEET_MYSQL_USERNAME'='fleet-user' \
  -e 'FLEET_MYSQL_PASSWORD'='fleet-user-password' \
  -e 'FLEET_REDIS_ADDRESS'='192.168.50.129:6379' \
  -e 'FLEET_OSQUERY_STATUS_LOG_PLUGIN'='filesystem' \
  -e 'FLEET_OSQUERY_RESULT_LOG_PLUGIN'='filesystem' \
  -e 'FLEET_FILESYSTEM_STATUS_LOG_FILE'='/mnt/data/logs/status.log' \
  -e 'FLEET_FILESYSTEM_RESULT_LOG_FILE'='/mnt/data/logs/result.log' \
  -e 'FLEET_FILESYSTEM_AUDIT_LOG_FILE'='/mnt/data/logs/audit.log' \
  -e 'FLEET_FILESYSTEM_ENABLE_LOG_ROTATION'='true' \
  -e 'FLEET_ACTIVITY_ENABLE_AUDIT_LOG'='true' \
  -e 'FLEET_ACTIVITY_AUDIT_LOG_PLUGIN'='filesystem' \
  -e 'FLEET_LOGGING_DEBUG'='true' \
  -e 'FLEET_SERVER_TLS'='false' \
  -p '8083:8080/tcp' \
  -p '8443:443/tcp' \
  -v '/mnt/user/appdata/fleetdm':'/mnt/data':'rw' \
  'fleetdm/fleet:latest' sh \
  -c "fleet prepare db && fleet serve"
```

To monitor the logs, especially for the Fleet server, you can run:

```bash
docker logs -f fleetdm
```

Look for messages indicating successful database migrations and that the server has started.

### Step 5: Accessing the Fleet UI
Once the containers are running, the Fleet UI should be accessible. Open a web browser and navigate to:
> http://<YOUR_DOCKER_HOST_IP>:8083

The first time you access Fleet, a setup screen will appear to create an administrator account. Follow the on-screen prompts. After setting up the admin user, you will be presented with the Fleet dashboard.

## BONUS ROUND: Enroll Your First Osquery Agent!
A Fleet server is most useful when it has agents reporting in. You can run an osquery agent as another Docker container.

1.  **Obtain Enroll Secret:**
    *   In the Fleet UI, navigate to **Hosts** and click the **Add hosts** button.
    *   Fleet will display an enroll secret. Copy this secret for the next step.

2.  **Run the Osquery Agent Container:**
    Open a terminal and execute the following command.
    - Replace `YOUR_ENROLL_SECRET_FROM_FLEET_UI` with the actual secret from the UI.
    - Replace `192.168.50.129` with your Docker host's IP address.

```bash
ENROLL_SECRET="YOUR_ENROLL_SECRET_FROM_FLEET_UI"
FLEET_URL="192.168.50.129:8083"

docker run -d --name osquery-agent-test \
  osquery/osquery:latest osqueryd \
  --enroll_secret=${ENROLL_SECRET} \
  --tls_hostname=${FLEET_URL} \
  --enroll_tls_endpoint=/api/osquery/enroll \
  --config_plugin=tls \
  --config_tls_endpoint=/api/osquery/config \
  --logger_plugin=tls \
  --logger_tls_endpoint=/api/osquery/log \
  --disable_distributed=false \
  --distributed_plugin=tls \
  --distributed_tls_read_endpoint=/api/osquery/distributed/read \
  --distributed_tls_write_endpoint=/api/osquery/distributed/write \
  --host_identifier=uuid \
  --force=true \
  --verbose=false \
  --tls_allow_unsafe=true
```

After a few moments, the new host should appear on the **Hosts** page in the Fleet UI.

## Cleaning Up: Returning to Shore
When you're finished exploring, you can cleanly remove the entire Fleet stack.

1.  **Stop the Containers:**
    ```bash
    docker stop MySQL Redis fleetdm osquery-agent-test
    ```

2.  **Remove the Containers:**
    ```bash
    docker rm MySQL Redis fleetdm osquery-agent-test
    ```

3.  **(Optional) Remove Host Directories:**
    If you want to remove all persisted data, you can delete the directories you created earlier:
    ```bash
    rm -rf /mnt/user/appdata/mysql
    rm -rf /mnt/user/appdata/fleetdm
    ```

And just like that, the system is back to its original state. This ease of teardown is invaluable for experimentation.

## Teaser for Part 2 - S3 Storage!
Feeling adventurous? Ready to take this Fleet setup to the next frontier? This Dockerized deployment is fantastic for getting started, but for more persistent or larger-scale scenarios, especially involving log retention and file carving, a more robust storage solution is beneficial.

**Stay Tuned!** In the next article in this series, the Fleet deployment will be supercharged by configuring **S3-compatible object storage (using MinIO, all in Docker, of course!)** for scalable log and file carve storage. Imagine effortlessly storing and managing extensive endpoint data!

Catch ya on the next project!

Josh 🖖

> Photo by [Mohammad Rahmani](https://unsplash.com/@afgprogrammer?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com)