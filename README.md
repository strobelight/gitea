# gitea

Git server with named volumes, based on information provided in [gitea docker install doc](https://docs.gitea.io/en-us/install-with-docker/).

For ssh to work with gitea, you'll need a git user per [this document](https://docs.gitea.io/en-us/install-from-binary/#prepare-environment). You'll also need an ssh key generated for this user, a special script, and find out the uid/gid. For example, as root on the host:

```
mkdir -p /app/gitea
cat <<EOF > /app/gitea/gitea
ssh -p 222 -o StrictHostKeyChecking=no git@127.0.0.1 "SSH_ORIGINAL_COMMAND=\"\$SSH_ORIGINAL_COMMAND\" \$0 \$@"
EOF
chmod +x /app/gitea/gitea
useradd --system --shell /bin/bash --comment 'Git Version Control' --user-group --create-home git
passwd --lock git
su - git
ssh-keygen -t rsa -N "" -C "Gitea git user host key"  # save to ~/.ssh/id_rsa
echo "$(cat /home/git/.ssh/id_rsa.pub)" > ~/.ssh/authorized_keys
id  # note the uid and gid
exit
# could "grep git /etc/passwd" too
```

Edit the `docker-compose.yml` file for different passwords to use. Change the UID and GID to match those from the above git user.

Also of note, ssh port is 222 unless also changed in the yml file, but if you change it, you'll need to also change it in /app/gitea/gitea script on host.

## start
You'll need docker and docker-compose.

```
docker-compose up -d
```

## access
Access from the host is [localhost:3000](http://localhost:3000).  You could edit /etc/hosts and add an alias of gitea, `127.0.0.1 gitea`.

If first time, access the Explore button, scan the information for correctness, and click Install at bottom to complete the installation. If no error immediately appears, give it time to install.

Access from other containers in the same network (--network=gitea_gitea) is [gitea:3000](http://gitea:3000).

## stop
```
docker-compose stop
```

## Jenkins
For integration with gitea:

- Install jenkins gitea plugin
- Configure access to gitea (manage -> configure, scroll to gitea server section)
- Configure gitea repo webhook to jenkins

### Jenkins config
For Jenkins, install the gitea plugin, then you can configure it by scrolling to the **Gitea Servers** section. If running Jenkins and Gitea from containers, ensure both on the same network.

### Gitea webhook config
The webhook to Jenkins using gitea is similar to `http://jenkins:8080/gitea-webhook/` (if using jenkins in a container on the same network with hostname jenkins).

### Troubleshooting

- ensure correct credential being used for access (user/password or user/token)
- ensure correct urls being used
- webhook issues may need advanced gitea server setup on Jenkins 
- if using docker containers for gitea as well as jenkins, you may run into cross-site issues if the containers domains are not the same. You may be able to get around by checking the _Enable proxy compatibility_ box.
- generate token for use with jenkins

## starting over
Some steps if you just want to start from a clean slate. _WARNING! This will destroy user accounts and stored repos!._

```
docker-compose stop  # ensure stopped
docker ps -a | grep gitea  # note the container ids for the gitea webserver and database (mariadb is a mysql fork)
docker rm xxxx yyyy  # remove the containers for the webserver and database
docker volume ls | grep gitea  # list gitea volumes
docker volume rm gitea_web  # use matching from above statement
docker volume rm gitea_db  # 

```

## config
[Configuration Cheat Sheet](https://docs.gitea.io/en-us/config-cheat-sheet/)

`app.ini` configures gitea. 

If using docker, 

```
docker exec -it -u git gitea bash
cd /data/gitea/conf
vi app.ini
```

Here are changes I made:

```
# in [repository] section:
ENABLE_PUSH_CREATE_USER = true

# in [server] section:
DOMAIN           = gitea
SSH_DOMAIN       = gitea
ROOT_URL         = http://gitea:3000/
```

After changes, restart via `/app/gitea/gitea manager restart` or do `docker-compose stop; docker-compose up -d`.

Be consistent. If you changed to use url like `gitea:3000`, use that everywhere else where needed, like in Jenkins config too.

