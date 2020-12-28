# DNS Round Robin Testing
alpine version: `alpine:3.10`

- Ever since Docker Engine 1.11, we can have multiple containers on a created network respond to the same DNS address.
- Create a new virtual network (default bridge driver)
- Create two container from `elasticsearch:2` image
- Research and use `--network-alias search` when creating them to give them an additional DNS name to respond to
- Run `alpine nslookup search` with `--net` to see the two containers list for the same DNS name

## Commands ran

- `docker network create dude`
- `docker container run -d --net dude --net-alias search elasticsearch:2`
- `docker container run -d --net dude --net-alias search elasticsearch:2`
- `docker container run --rm --net dude alpine:3.10 nslookup search`
- `docker container run --rm --net dude centos curl -s search:9200` (this allows us to spin up a container running centos to execute a command and because of the `--rm` option it will clean itself up after its finished).
