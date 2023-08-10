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
6.I built the reverse proxy image upon this new conf file and pushed to my hub.docker.com as engyfouda/atseasampleshopapp_reverse_proxy.
I updated the docker-stack.yml with this new image.


These are the steps I followed to try the application Docker Playground:
1. git clone https://github.com/efoda/atsea-sample-shop-app
2. cd atsea-sample-shop-app/
3. docker swarm init
4. mkdir certs
5. openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
6.docker secret create revprox_cert certs/domain.crt
7. docker secret create revprox_key certs/domain.key
8. docker secret create postgres_password certs/domain.key
9. docker-compose up --build
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


