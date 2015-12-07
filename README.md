# FIWARE Docker Container Service


# Goal
The FIWARE Docker Container Service exposes the docker API so so that FIWARE tenants can use their docker client remotely to manage their containers.  However tenants are insulated from each other so they can only manage containers that they have created.
#Description
Docker Swarm is native clustering for Docker. It allows access to a pool of Docker hosts using the full suite of Docker tools. Because Docker Swarm serves the standard Docker API, any tool that already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts.  The service enhances Swarm to support multi-tenancy.
#Rational
The service will simplify the usage of docker to develop FIWARE applications since tenant administrators will be relieved of the task of managing their docker hosts; the docker hosts will be managed by the FIWARE Lab infrastructure providers.  Further more since the docker hosts and related resources will be shared by multiple tenants this will allow the FIWARE lab to achieve greater density of containers per host.
