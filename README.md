# gitea

Git server with named volumes, based on information provided in [gitea docker install doc](https://docs.gitea.io/en-us/install-with-docker/).

You will need a git user per [prepare environment](https://docs.gitea.io/en-us/install-from-binary/#prepare-environment). Find out the uid/gid. For example, as root on the host:

```
useradd --system --shell /bin/bash --comment 'Git Version Control' --user-group --create-home git
passwd --lock git
su git
id  # note the uid and gid
exit
# could "grep git /etc/passwd" too
```

Edit the `docker-compose.yml` file for different passwords to use. Change the UID and GID to match those from the above git user.

Also of note, ssh port is 222 unless also changed in the yml file.

## start

```
docker-compose up -d
```

## access
[localhost:3000](http://localhost:3000)

If first time, access the Explore button, scan the information for correctness, and click Install at bottom to complete the installation. If no error immediately appears, give it time to install.

## stop
```
docker-compose stop
```

## starting over
Some steps if you just want to start from a clean slate. _WARNING! This will destroy user accounts and stored repos!._

```
docker-compose stop  # ensure stopped
docker ps -a  # note the container ids for the gitea webserver and database (mariadb is a mysql fork)
docker rm xxxx yyyy  # remove the containers for the webserver and database
docker volume ls | grep gitea  # list gitea volumes
docker volume rm gitea_web  # use matching from above statement
docker volume rm gitea_db  # 

```
