---
title: Docker
tags: technology, devops
---

### Using *Docker for Mac* and *Docker Toolbox* simultaneously
When you want to use **Docker for Mac**, make sure all DOCKER\_\* environment variables are unset with `$ unset ${!DOCKER_*}`. When you want to run one of the {{VirtualBox}} VMs you have set with docker-machine, run `eval $(docker-machine env NAME)`.

### Rundown
1. Create new machines with `$ docker-machine create —driver virtualbox NAME`
1. Start/stop machines with `$ docker-machine start/stop`
1. Establish env variables to point `docker` to that machine with `$ eval $(docker-machine env NAME)`
1. Run images, start/stop or see processes with `$ docker ps/start/stop/run`

### Useful Aliases and Bash Functions
`
docker-switch () { docker-machine start "$1" && eval $(docker-machine env "$1"); }
`
