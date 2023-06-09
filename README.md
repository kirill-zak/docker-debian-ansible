# docker-debian-ansible
Debian docker container with built-in Ansible for playbook and role testing

# The main goal
Keep all version in one repository under different tags

## Tags
- `10`. Based on Debian 10 (Buster)
- `11`. Based on Debian 11 (Bullseye)
- `12`. Based on Debain 12 (Bookworm)

## How to Build

To build the image on your own locally, do the following:

  1. [Install Docker](https://docs.docker.com/engine/installation/).
  2. `cd` into target directory. For example, target directory for `Debian 12` is `debian12`
  3. Run `docker build -t docker-debian-ansible .`.

  ## How to Use

  1. [Install Docker](https://docs.docker.com/engine/installation/).
  2. Pull pre-build image via `docker image pull kirillzak/docker-debian-ansible:latest` or build image.
  3. Run a container from the image: `docker run --detach --privileged --volume /sys/fs/cgroup:/sys/fs/cgroup:ro docker-debian-ansible:latest`

## Author

[Kirill Ziuzin](https://kirill-zak.ru/)

### Thanks
Thanks for [Jeff Geerling](https://github.com/geerlingguy)

### Based
Based on:
- https://github.com/geerlingguy/docker-debian10-ansible
- https://github.com/geerlingguy/docker-debian11-ansible
