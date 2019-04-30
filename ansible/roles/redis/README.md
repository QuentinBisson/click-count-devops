Role Name
=========

Start a Redis server as a docker container.

Requirements
------------

- docker must be installed

Role Variables
--------------

redis_container_image (redis:5.0.4-alpine): Redis image
redis_container_name (redis): Redis container rname
redis_container_network (redis-network): Redis container network
redis_container_storage_location (/srv/redis/data): Default location for redis data on the host

License
-------

GPLv3