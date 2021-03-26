# gitea

Git server with named volumes, based on information provided in [gitea docker install doc](https://docs.gitea.io/en-us/install-with-docker/).

Edit the `docker-compose.yml` file for different passwords to use.

Also of note, ssh port is 222 unless also changed in the yml file.

## start

```
docker-compose up -d
```

## access
[localhost:3000](http://localhost:3000)

## stop
```
docker-compose stop
```
