Role Name
=========

A role to install Docker.

Role Variables
--------------

docker_package (docker-ce): The docker package to install ()
docker_apt_key: Docker GPG Key url
docker_apt_release_channel (stable): Docker release channel
docker_apt_arch (amd64): Host architecture
docker_apt_repository: Docker apt repository url

docker_registry_url: Docker registry url
docker_registry_username: Docker registry user

docker_user: User running docker containers

License
-------

GPLv3