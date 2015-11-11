## Jira Server in Docker
[Docker image](https://hub.docker.com/r/million12/jira) with [Atlassian Jira](https://www.atlassian.com/software/jira) build in docker. Integrated with MySQL/MariaDB support. Base image is [million12/centos-supervisor](https://hub.docker.com/r/million12/centos-supervisor/) which is based on offcial CentOS-7 image.  

User will need a licence to be able to finish setup. Either evaluation one or full.

This image can be run just by itself but then it will have support only for Jira build-in database HSQL.

## Basic Usage

    docker run \
    -d \
    --name jira \
    -p 8080:8080 \
    million12/jira

### Setup
Access setup page under [docker.ip:8080]() and follow installation procedure.  
**Atlassian Jira licence is required to finish installation.**

## ENVIRONEMNTAL VARIABLES
This image comes with few environmental variables that will be needed to connect it to MYSQL/MariaDB container.  

`DB_SUPPORT` - Enabling certain database support. (`mysql` or `mariadb` support at the moment. PostgerSQL will be implemented in the future).  
`MARIADB_USER` - Database administrator username.  
`MARIADB_PASS` - Database administrator password.  
`JIRA_DB_ADDRESS` - Database address (ip or domain.com format).  
`JIRA_DB_NAME` - Jira database name.  
`JIRA_USER` - Jira database username.  
`JIRA_PASS` - Jira database password.  

## MySQL/MariaDB and Data Container Usage
In this example we will deploy Jira server with MySQL/MariaDB support and with separate Data container which will keep installation files and Jira user-data files shared with host os for easy backup.

### MariaDB Docker image
    docker run -d \
      --name jira-db \
      -e MARIADB_USER=admin \
      -e MARIADB_PASS=password \
      million12/mariadb

### Data container

    docker run -d \
      --name jira-data \
      -v /data/jira/jira-install:/opt/atlassian \
      -v /data/jira/jira-data:/var/atlassian \
      busybox:latest

### Jira container

    docker run -d \
      --name jira \
      --link jira-db:jira.db \
      --volumes-from jira-data \
      -e MARIADB_USER=admin \
      -e MARIADB_PASS=password \
      -e JIRA_DB_ADDRESS=jira.db \
      -e JIRA_DB_NAME=jiradb \
      -e JIRA_USER=jira-user \
      -e JIRA_PASS=password \
      -e DB_SUPPORT=mariadb \
      -p 80:8080 \
      million12/jira

### Docker troubleshooting


Use docker command to see if all required containers are up and running:

    $ docker ps -a

Check online logs of jira container:

    $ docker logs jira

Attach to running jira container (to detach the tty without exiting the shell,
use the escape sequence Ctrl+p + Ctrl+q):

    $ docker attach jira

Sometimes you might just want to review how things are deployed inside a running container, you can do this by executing a _bash shell_ through _docker's exec_ command:

    docker exec -i -t jira /bin/bash

History of an image and size of layers:

    docker history --no-trunc=true million12/jira | tr -s ' ' | tail -n+2 | awk -F " ago " '{print $2}'

---
## Author

Author: Przemyslaw Ozgo [linux@ozgo.info](mailto:linux@ozgo.info)

---

**Sponsored by** [Typostrap.io - the new prototyping tool](http://typostrap.io/) for building highly-interactive prototypes of your website or web app. Built on top of TYPO3 Neos CMS and Zurb Foundation framework.
