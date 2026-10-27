docker build -f logger-service.dockerfile -t gabangel/logger-service:1.0.0 .
docker push gabangel/logger-service:1.0.0

`to login`
docker login



init as part of node: `docker swarm init`
deploy: `docker stack deploy -c swarm.yml microservice`
list services: `docker swervice ls`