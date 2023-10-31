# Docker and PHP Complete Tutorial

YouTube: https://www.youtube.com/watch?v=qv-P_rPFw4c&t=2s

## Chapter One - Creating a Docker Compose file

Docker compose file typically named `docker-compose.yaml` is a file docker uses to create a network of containers. You create the file and list the services, or containers, desired for your application to work. The normal convention is to name services after the specific use in the application rather than the actual service its providing. For example, NGINX containers can often be called a web service because you're using that NGINX container to serve your web app.

We'll start with NGINX service but in order to actually see any progress you have to map the port in the container to a port on your machine. In this case, we'll map port 80 locally, to port 80 in the container.

Once that's done you actually have all you need to spin up a container and view it locally on `localhost:80`. Just use the `docker-compose up` command. You will be able to view the output as the container is starting since we didn't add the `-d` flag.