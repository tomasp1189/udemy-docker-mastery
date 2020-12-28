# Assignment: CLI App Testing

- Use different linux distro containers to check `curl` cli tool version
- Use two different terminal windows to start bash in both `centos:7` and `ubunto:14.04`, using `-it`.
- Learn the `docker container --rm` option so you can save cleanup
- Ensure `curl` is installed and on latest version for the distro
  - ubunto: `apt-get update && apt-get install curl`
  - centos: `yum update curl`
- Check `curl --version`

## Commands ran
### centos
- `docker container run --rm --it centos:7 bash`
- `yum udpate curl`
- `curl --verion`
### ubuntu
- `docker container run --rm -it ubuntu:14.04 bash`
- `apt-get update && apt-get install -y curl`
- `curl --verion`
