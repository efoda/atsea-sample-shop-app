**My edits on the original app:**
1. In the Dockerfiles, I changed the image Java to JDK because Java is depricated.
2. In docker-compose.yml, I changed the specifications of the network: payemt and added "attachable: true" because I recevied an error that it's not attachable by default.
3. In the Dockerfile of the app service, I added "RUN groupadd -r gordon && useradd -r -g gordon gordon
USER gordon" to resolve the user and passwrord errors.
4. In the Dockerfile of the database service, I hashed the configuration files to disable the monitoring because it generates "incomplete startup packets" for the postgres. However, it didn't work. The containers of the app and the database exit instantly after being up and running.
5. I was getting in the container logs that "The ssl directive is deprecated," so I removed it from ./reverse-proxy/nginx.conf file.
Then I got the error message that "the plain http request was sent to https port nginx."
I added some proxy_host overrides to avoid this message. HOwever, the error message persisted.
Therefore, I changed the port 443 to 81 because 443 is an HTTPS port. It works now elhamdleAllah!
6. I built the reverse proxy image upon this new conf file and pushed to my hub.docker.com as engyfouda/atseasampleshopapp_reverse_proxy.
I updated the docker-stack.yml with this new image.
7. I updated the reverse_proxy service in the docker-stack.yml file with my working image, engyfouda/atseasampleshopapp_reverse_proxy, and changed the HTTPS port to HTTP port.
8. Also, when use this app with Docker UCP, there was a conflict in the ports as both use port 443 at the host. Therefore I changed the host post to 8002 and the container port to 81 which is an HTTP port, not HTTPS port in the docker-stack.yml file.
9. I updated aall the images that are in the docker-stack.yml with images that I built and pushed to my hub.docker.com account as many of these images sometimes get moved or removed or not able to download them because of the vulnerabilities on the official images.
10. In the /app/Dockerfile, I edited the node image to one that I pushed to my hub account as the official node image was removed. Therefore, trying docker-compose up was failing when building this image because docker daemon cant pull the official image.


**These are the steps I followed to try the application Docker Playground:**

1.Create two nodes and connect them on a swarm. Run "$ docker swarm init" on the first node. Copy the generated token and run it on the second node.

2. git clone https://github.com/efoda/atsea-sample-shop-app

3. cd atsea-sample-shop-app/

4. mkdir certs

5. openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

6.docker secret create revprox_cert certs/domain.crt

7. docker secret create revprox_key certs/domain.key

8. docker secret create postgres_password certs/domain.key

9. echo staging | docker secret create staging_token - 

10. docker stack deploy -c docker-stack.yml atsea
11. connect to http://localhost:8002 or the http://<UCP node IP>:8002
    Note: please wait a couple of minutes and the keep refreshing. At first, it'll give 502 bad gateway, but after few seconds the Atsea store will work.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------

![](atsea_store.png)
#  AtSea Shop Demonstration Application

The AtSea Shop is a demonstration application comprised of: 

* Java REST application written using Spring-Boot, 
* a database for product inventory, customer data, and orders,
* a React shopping cart,
* a NGINX reverse proxy implementing https,
* a payment gateway to simulate certificate management

# Requirements

This example uses features in Docker 17.05 CE Edge. Install this version to run the example.

# Building and Running the AtSea Shop

## Secrets

This application uses Docker secrets to secure the application components. The reverse proxy requires creating a certificate that is stored as a secret and the payment also requires a password stored as a secret. To create a certificate and add as a secret:

```
mkdir certs

openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

docker secret create revprox_cert certs/domain.crt

docker secret create revprox_key certs/domain.key

docker secret create postgres_password certs/domain.key
```

To create a secret for staging the payment gateway:

```
echo staging | docker secret create staging_token - 
```

## Run as an application

To run the AtSea shop as an application:
```
docker-compose up --build
```

## Deploy to a swarm
```
#If you need to create a Swarm
docker swarm init
docker stack deploy -c docker-stack.yml atsea
```

## A simplified development environment
This compose file creates a simplified development environment consisting of only the application server and the database.

```
docker-compose --file docker-compose-dev.yml up --build
```



## The AtSea Shop 

The URL for the content is `http://localhost:8080/`

# REST API

Documentation for REST calls: [REST API](./REST.md)


