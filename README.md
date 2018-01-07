# Docker Website Template

A Docker template based on the [docker-nginx-letsencrypt-sample](https://github.com/gilyes/docker-nginx-letsencrypt-sample) project.

This creates the following containers:

1. An nginx proxy in front of all websites
1. A lets encrypt client that automatically creates certificates for all proxied websites and updates the nginx proxy
1. A nginx-generator that detects any addition of new websites to the docker -compose setup
1. Any custom websites you specify in the docker-compose file.

In the example setup the website called sample-website and is configured in the following places:

1. volumes/nginx-sample-website/conf.d/sample-website.conf - __nginx conf__
1. volumes/nginx-sample-website/content - __web root dir__

Copy any files you need served into the __volumes/nginx-sample-website/content__ directory.

Every website added should have the following configuration set:

```
    volumes:
      - "./volumes/your_website/conf.d/:/etc/nginx/conf.d"
      - "./volumes/your_website/content/:/usr/share/nginx/html/"
    environment:
      - VIRTUAL_HOST=your_website,www.your_website
      - VIRTUAL_NETWORK=nginx-proxy
      - VIRTUAL_PORT=80
      - LETSENCRYPT_HOST=your_website,www.your_website
      - LETSENCRYPT_EMAIL=your_email@something.com
```

__Remember__ to match the VIRTUAL_HOST and LETSENCRYPT_HOST used with those specified in the _server_name_ attribute of your config file at __volumes/your_website/conf.d/your.conf__

## Config Changes

1. Update __VIRTUAL_HOST__ and __LETSENCRYPT_HOST__ with your domain names within the docker-compose file.
1. ./volumes/your_website/conf.d/sample-website.conf with your domain names
1. Update __LETSENCRYPT_EMAIL__ with your email address within the docker-compose file.

## Running
In the main directory run:
```bash
docker-compose up
```

## ReRunning
1. Remove all containers from /app dir - removes stale data inside the containers such as letencrypt config:
```
docker-compose down
```

2. Remove all docker images for Dockerfiles that have changed as this will force a rebuild of the image:
```
docker image rm name
```

3. Remove all volumes to ensure updated config is detected:
```
docker volume ls | awk '{if(NR>1)print}' | tr -s " " | cut -d " " -f2 | xargs docker volume rm
```
If you can't remove some volumes because they are being used, you can either remove the hashes of the containers using the volumes with:

```
docker stop HASH
docker rm HASH
```

or remove all containers with:

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

## Testing
1. To use the [letsencrypt staging environment](https://letsencrypt.org/docs/staging-environment/) set the __ACME_CA_URI__ environment variable in the docker-compose.yml file under the __letsencrypt-nginx-proxy-companion__ section for environment vars: ```ACME_CA_URI=https://acme-staging.api.letsencrypt.org/directory```
1. To add debug logging to the letsencrypt container add the fllowing environment var: ```DEBUG=true```

## Troubleshooting
* To view logs run `docker-compose logs`.
* To view the generated Nginx configuration run `docker exec -ti nginx cat /etc/nginx/conf.d/default.conf`
* To stop and remove all containers: ```docker-compose down```


