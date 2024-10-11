---
myst:
  html_meta:
    description: A cheatsheet of useful commands that are available through SSH.
    title: Hypernode commands cheatsheet.
redirect_from:
  - /en/hypernode/ssh/cheatsheet/
  - /knowledgebase/ssh-cheatsheet/
---

This document is meant to serve as a reference reference manual for Hypernode usage through SSH.
It also functions as a method of documenting our custom commands in a more organised way.



# PNL (Parse-Nginx-Log) Reference.
PNL Is a JSON parser. It's meant to be used to make nginx logs easier to read and filter.
Here are a few examples that I use very often as a support engineer at Hypernode.


## List the top 5 bots and the amount of requests sent to the node yesterday & today.
```
pnl --yesterday --php --bots --fields ua | sort | uniq -c | sort -n | tail -n 5; \
pnl --today --php --bots --fields ua | sort | uniq -c | sort -n | tail -n 5;
```

## You may also use a loop to go through more than 1 day in 1 command: 
### Show last 7 days of bot traffic by day.
```
for i in {00..6}; do
    printf "Days ago: $i \n"
    if [ "$i" -eq 0 ]; then
        hypernode-parse-nginx-log --today --php --bots --fields ua | sort | uniq -c | sort -n | tail -n 5
        printf "\n"
    else
        hypernode-parse-nginx-log --days-ago "$i" --php --bots --fields ua | sort | uniq -c | sort -n | tail -n 5
        printf "\n"
    fi
done
```

### List the 5 IP addresses that sent the most requests to the server.
```
pnl --today  --fields ip | sort | uniq -c | sort -n | tail -n 5
```
### List the 10 countries that sent the most requests to your node today. Sorted by amount of requests.
```
pnl --today --fields country | sus | tail -10
```

### List amount of requests per IP for a url containing the word 'image', sorted by amount of requests today. 
```
pnl --today --fields remote_addr,req --filter req~image | sort | uniq |
    cut -d' ' -f1 | sort | uniq -c | sort -n | grep -v ' 1 ' | tail -n 5;
```

### Embedded documentation
```
usage: hypernode-parse-nginx-log [-h] [--fields FIELDS] [--php] [--bots] [--format FORMAT] [--list-fields] [--filter FILTER] [--verbose] [--ncsa] [--filename FILENAME | --today | --yesterday | --days-ago DAYS_AGO | --date DATE]

options:
  -h, --help           show this help message and exit

  --fields FIELDS      Comma separated list of fields to display. Available fields: body_bytes_sent, user_agent, time, port, remote_addr, status, handler, host, country, referer, server_name, request_time, ssl_cipher, ssl_protocol, remote_user, request . 
  Use --list-fields to display an updated list of all available fields.

  --php                Short for --filter handler~phpfpm
  
  --bots               Short for --filter user_agent~(http|bot|crawl|spider|search)

  --format FORMAT      Format string to display fields

  --list-fields        Display a list of available fields

  --filter FILTER      Filter to apply. Format: <field>=<str> or <field>~<regex> or <field>!~<regex>

  --verbose, -v        Display debug output

  --ncsa, --apache     Output in NCSA format.
  
  --filename FILENAME  Path of nginx logfile to parse
  
  --today              Analyze logs and outputs today's log lines
  
  --yesterday          Analyze logs and outputs yesterday's log lines
  
  --days-ago DAYS_AGO  Analyze logs and outputs for a specific number of days ago
  
  --date DATE          Analyze logs and outputs for a specific date
```



# Hypernode-Manage-Vhosts \ HMV
 
## Enabling Varnish
```
hmv [VHOST] --varnish
```

## Request free SSL certificates using LetsEncrypt & force HTTPS
```
hmv --all --https --force-https
```

## Manually set webroot directory
```
hmv [VHOST/Domain] --webroot /data/web/exampledirectory
```

## Embedded documentation
```
usage: main.py [-h] [--all] [-v] [--varnish] [--default-server]
               [--type {magento2,vuestorefront,ab-test,shopware5,wordpress,shopware6,pwa-studio,psb,magento1,akeneo4,mailhog,generic-php,akeneo,wwwizer}] [--webroot WEBROOT]
               [--delete] [--skip-event] [--handler {phpfpm}] [-y | -n] [--https] [--force-https] [--ssl-config {modern,intermediate}] [--ssl-noclobber] [--port-http PORT_HTTP]
               [--port-https PORT_HTTPS] [--port-http-staging PORT_HTTP_STAGING] [--port-https-staging PORT_HTTPS_STAGING] [--list [servername ...]] [--format {table,json,raw}]
               [--import-statefile IMPORT_STATEFILE] [--protected-files PROTECTED_FILES [PROTECTED_FILES ...]] [--import-dir IMPORT_DIR]
               [servernames ...]

Manage the nginx config for one or more given servernames

positional arguments:
  servernames           Servernames to manage the root nginx config for

options:
  -h, --help            show this help message and exit
  --all                 Apply the configuration for all vhosts defined in the statefile.
  -v, --verbose         Be more verbose
  --varnish, --disable-varnish
                        Manage varnish for the servername
  --default-server      Use this servername as default server
  --type {magento2,vuestorefront,ab-test,shopware5,wordpress,shopware6,pwa-studio,psb,magento1,akeneo4,mailhog,generic-php,akeneo,wwwizer}
                        Create configuration based on the given type
  --webroot WEBROOT     Specify the location of the webroot directory of your installation. This defaults to values specific for the given type
  --delete              Delete the servername from the root nginx config
  --skip-event          Don't send event to the NATS event bus
  --handler {phpfpm}    Specify the php handler you want to assign. If not specified the main php handler will be used
  -y, --yes             Answer yes on any confirmation requests
  -n, --no              Answer no on any confirmation requests

SSL related configs:
  --https, --disable-https
                        Sets the use of https. Will create and use a letsencrypt certificate if there is no other SSL certificate present
  --force-https, --disable-force-https
                        Force http to be redirected to https
  --ssl-config {modern,intermediate}
                        Use the given Mozilla SSL standard config
  --ssl-noclobber       Do not overwrite existing SSL certificate

Listen ports:
  --port-http PORT_HTTP
                        Port that listens to http traffic
  --port-https PORT_HTTPS
                        Port that listens to https traffic
  --port-http-staging PORT_HTTP_STAGING
                        Port that listens to http staging traffic
  --port-https-staging PORT_HTTPS_STAGING
                        Port that listens to https staging traffic

Information about active VHosts:
  --list [servername ...]
                        Servernames to list. Will show all managed VHosts if none given
  --format {table,json,raw}
                        What way should the output be listed

Importer:
  --import-statefile IMPORT_STATEFILE
                        Location of the imported statefile
  --protected-files PROTECTED_FILES [PROTECTED_FILES ...]
                        Files to protect: these will not be overwritten
  --import-dir IMPORT_DIR
                        Location of the imported nginx directory
```

# Hypernode-systemctl
## A tool which allows you to use specific root commands as a non-root user.
```
usage: hypernode-systemctl [-h] [--verbose]
                           {settings,alternative_php_versions,show,whitelist,block_attack,xgrade,autoscaling,list_xgrades,attach_backup,list_backups,create_backup,preinstall,brancher}
```

## List enabled alternative PHP Versions.
```
hypernode-systemctl alternative_php_versions list
```

```
## Enable alternative PHP Version.
hypernode-systemctl alternative_php_versions [Version | eg. 8.0]
```

## Manually create a backup. Only available if you are on the standard SLA. (Not on basic)
```
hypernode-systemctl create_backup
> Create backup job posted, see hypernode-log (or livelog) for job progress. This backup can be attached by using the 'hypernode-systemctl attach_backup' command. 
```

# NGINX configuration
## A more technical overview of Hypernode's Nginx implementation.
Additional context: https://docs.hypernode.com/hypernode-platform/nginx/how-to-use-nginx.html#inclusion-order

### Files/context blocks.
Can only be placed in the /data/web/nginx directory. Logic in any http.* file applies to all managed vhosts unless it is overwritten by a file that is loaded later.
```
http.* 
```

May be placed in a subdirectory to make the logic only apply for a specific vHost.
```
server.*
```

Any files using this naming scheme will apply to the staging instance of a specified vHost.
You specify the vhost the logic applies to by deciding which subdirectory you place the file in.
```
staging.*
```

Opposite of staging.*
```
public.*
```