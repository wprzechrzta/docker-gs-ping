

# docker-gs-ping
1. Build
docker build --tag docker-gs-ping .
docker build -t docker-gs-ping:multistage -f Dockerfile.multistage .
docker image ls
docker image rm <imageId>

2. Run
docker run -p 8080:8080 docker-gs-ping
docker ps --all
docker stop <contId>

docker restart <contId>
docker ps -a

// run as daemon
docker run -d -p 8080:8080 --name rest-server docker-gs-ping

3. Run with database (cockroachdb)
docker volume create roach
docker network create -d bridge mynet

3.1 start db
docker run -d \
--name roach \
--hostname db \
--network mynet \
-p 26257:26257 \
-p 8080:8080 \
-v roach:/cockroach/cockroach-data \
cockroachdb/cockroach:latest-v20.1 start-single-node \
--insecure

3.2 Init db and user
docker exec -it roach ./cockroach sql --insecure
CREATE DATABASE mydb;
CREATE USER totoro;
GRANT ALL ON DATABASE mydb TO totoro;
quit;

3.3 Start example app
gclo git@github.com:wprzechrzta/docker-gs-ping-roach.git

3.4 Build
docker build --tag docker-gs-ping-roach .

3.5 Run  and connect to cockroach
   docker run -it --rm -d \
   --network mynet \
   --name rest-server \
   -p 80:8080 \
   -e PGUSER=totoro \
   -e PGPASSWORD=myfriend \
   -e PGHOST=db \
   -e PGPORT=26257 \
   -e PGDATABASE=mydb \
   docker-gs-ping-roach

curl localhost
//docker container rm --force rest-server

curl --request POST \
--url http://localhost/send \
--header 'content-type: application/json' \
--data '{"value": "Hello, Docker!"}'
 
or 

curl -X POST \
--header 'content-type: application/json' \
--data '{"value": "Hello, Second message!"}' \
http://localhost/send 


# Doc
A simple Go server/microservice example for [Docker's Go Language Guide](https://docs.docker.com/language/golang/).

Notable features:

* Includes a [multi-stage `Dockerfile`](https://github.com/olliefr/docker-gs-ping/blob/main/Dockerfile.multistage), which actually is a good example of how to build Go binaries _for production releases_.
* Has functional tests for application's business requirements with proper isolation between tests using [`ory/dockertest`](https://github.com/ory/dockertest).
* Has a CI pipeline using GitHub Actions to run functional tests in independent containers.
* Has a CD pipeline using GitHub Actions to publish to Docker Hub.

## Want _moar_?!
There is a more advanced example in [olliefr/docker-gs-ping-roach](https://github.com/olliefr/docker-gs-ping-roach) using [CockroachDB](https://github.com/cockroachdb/cockroach).

## License

[Apache-2.0 License](LICENSE)
