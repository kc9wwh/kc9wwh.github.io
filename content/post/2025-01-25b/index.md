---
title: "Don't Forget to Apply Updates to your DB"
description: "Why putting off those db migration warnings is a bad idea.."
date: 2025-01-26T08:33:38Z
image: cover.jpg
categories: [Tech]
tags: [Fleet, Open Source, MySQL]
hidden: false
comments: true
draft: false
---
About a month ago I setup [Fleet Device Management](https://fleetdm.com) and since then there have been a couple updates that of come out and along with that some required database migrations or updates that need to happen. Unlike with other MDM's I've used in the past, the migrations that need to happen on the database do not auto apply when an upgraded version is ran against it. Instead, you need to manually complete them. 

Now I've see these warning for a couple weeks, but kept putting them off. Until today at least, as after updating my fleetdm/fleet docker container to the latest version, Fleet would no longer startup. *ruh-roh!*

What the first thing we do when a server doesn't boot, we try again...only, that didn't work.

Whats the second thing we do? That's right! We check the logs and those logs were all about telling me I really need to do these database migrations. But how since the docker container immediately stops after starting it?

First we need to understand why the container keeps stopping and that is cause unless it is doing something (i.e., a service is actively running on the container), docker, will stop the container to save on resources being wasted. So, how do we overcome this when the container isn't doing what we *want* it to do? We give it something else to do, by having it run tail against `/dev/null`. 

**Normal docker run Command:**
```
docker run
  -d
  --name='fleetdm'
  --net='bridge'
  --pids-limit 2048
  -e TZ="America/Chicago"
  -e HOST_HOSTNAME="Loki"
  -e HOST_CONTAINERNAME="fleetdm"
  -e 'FLEET_MYSQL_ADDRESS'='10.0.10.82:3306'
  -e 'FLEET_MYSQL_DATABASE'='YOUR_FLEET_DB'
  -e 'FLEET_MYSQL_USERNAME'='YOUR_FLEET_USER'
  -e 'FLEET_MYSQL_PASSWORD'='YOUR_FLEET_PASS'
  -e 'FLEET_REDIS_ADDRESS'='192.168.50.129:6379'
  -e 'FLEET_SERVER_CERT'='/mnt/data/certs/certificate.crt'
  -e 'FLEET_SERVER_KEY'='/mnt/data/certs/private.key'
  -e 'FLEET_MDM_WINDOWS_WSTEP_IDENTITY_CERT'='/mnt/data/win_mdm_certs/fleet-mdm-win-wstep.crt'
  -e 'FLEET_MDM_WINDOWS_WSTEP_IDENTITY_KEY'='/mnt/data/win_mdm_certs/fleet-mdm-win-wstep.key'
  -e 'FLEET_SERVER_PRIVATE_KEY'='YOUR_SERVER_PRIVATE_KEY'
  -e 'FLEET_LICENSE_KEY'='YOUR_KEY_HERE'
  -e 'FLEET_OSQUERY_STATUS_LOG_PLUGIN'='filesystem'
  -e 'FLEET_OSQUERY_RESULT_LOG_PLUGIN'='filesystem'
  -e 'FLEET_FILESYSTEM_STATUS_LOG_FILE'='/mnt/data/logs/status.log'
  -e 'FLEET_FILESYSTEM_RESULT_LOG_FILE'='/mnt/data/logs/result.log'
  -e 'FLEET_FILESYSTEM_AUDIT_LOG_FILE'='/mnt/data/logs/audit.log'
  -e 'FLEET_FILESYSTEM_ENABLE_LOG_ROTATION'='true'
  -e 'FLEET_ACTIVITY_ENABLE_AUDIT_LOG'='true'
  -e 'FLEET_ACTIVITY_AUDIT_LOG_PLUGIN'='filesystem'
  -e 'FLEET_LOGGING_DEBUG'='true'
  -p '8081:8080/tcp'
  -v '/mnt/user/appdata/fleetdm':'/mnt/data':'rw' 'fleetdm/fleet'
```

**Normal docker run w/ Tail Command:**
```
docker run
  -d
  --name='fleetdm'
  --net='bridge'
  --pids-limit 2048
  -e TZ="America/Chicago"
  -e HOST_HOSTNAME="Loki"
  -e HOST_CONTAINERNAME="fleetdm"
  -e 'FLEET_MYSQL_ADDRESS'='10.0.10.82:3306'
  -e 'FLEET_MYSQL_DATABASE'='YOUR_FLEET_DB'
  -e 'FLEET_MYSQL_USERNAME'='YOUR_FLEET_USER'
  -e 'FLEET_MYSQL_PASSWORD'='YOUR_FLEET_PASS'
  -e 'FLEET_REDIS_ADDRESS'='192.168.50.129:6379'
  -e 'FLEET_SERVER_CERT'='/mnt/data/certs/certificate.crt'
  -e 'FLEET_SERVER_KEY'='/mnt/data/certs/private.key'
  -e 'FLEET_MDM_WINDOWS_WSTEP_IDENTITY_CERT'='/mnt/data/win_mdm_certs/fleet-mdm-win-wstep.crt'
  -e 'FLEET_MDM_WINDOWS_WSTEP_IDENTITY_KEY'='/mnt/data/win_mdm_certs/fleet-mdm-win-wstep.key'
  -e 'FLEET_SERVER_PRIVATE_KEY'='YOUR_SERVER_PRIVATE_KEY'
  -e 'FLEET_LICENSE_KEY'='YOUR_KEY_HERE'
  -e 'FLEET_OSQUERY_STATUS_LOG_PLUGIN'='filesystem'
  -e 'FLEET_OSQUERY_RESULT_LOG_PLUGIN'='filesystem'
  -e 'FLEET_FILESYSTEM_STATUS_LOG_FILE'='/mnt/data/logs/status.log'
  -e 'FLEET_FILESYSTEM_RESULT_LOG_FILE'='/mnt/data/logs/result.log'
  -e 'FLEET_FILESYSTEM_AUDIT_LOG_FILE'='/mnt/data/logs/audit.log'
  -e 'FLEET_FILESYSTEM_ENABLE_LOG_ROTATION'='true'
  -e 'FLEET_ACTIVITY_ENABLE_AUDIT_LOG'='true'
  -e 'FLEET_ACTIVITY_AUDIT_LOG_PLUGIN'='filesystem'
  -e 'FLEET_LOGGING_DEBUG'='true'
  -p '8081:8080/tcp'
  -v '/mnt/user/appdata/fleetdm':'/mnt/data':'rw' 'fleetdm/fleet'
  tail -f /dev/null
```

Now that I gave the container something to do, besides starting Fleet since that is immediately stopping, I can now pull up an interactive terminal and perform the database migrations. 
```
docker exec -it fleetdm fleet prepare db
```

You could also run this as a non-interactive command using `docker exec fleetdm fleet prepare db`, but as it was the first time running this for an upgrade, I wanted to be able to walk through it without the terminal session suddenly closing. 

That was it! Pretty painless once you know what you're doing, but definitely something you need to be aware of when performing upgrades to Fleet and to always check if there are any pending db migrations that need to occur after. 


Catch ya on the next project!

Josh 🖖

> Photo by [Fotis Fotopoulos](https://unsplash.com/@ffstop?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com)