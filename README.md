# Docker and PHP Complete Tutorial

YouTube: https://www.youtube.com/watch?v=qv-P_rPFw4c&t=2s

## Chapter  One - Creating a Docker Compose file

Docker compose file typically named `docker-compose.yaml` is a file docker uses to create a network of containers. You create the file and list the services, or containers, desired for your application to work. The normal convention is to name services after the specific use in the application rather than the actual service its providing. For example, nginx containers can often be called a web service because you're using that nginx container to serve your web app.

We'll start with nginx service but in order to actually see any progress you have to map the port in the container to a port on your machine. In this case, we'll map port 80 locally, to port 80 in the container.

Once that's done you actually have all you need to spin up a container and view it locally on `localhost:80`. Just use the `docker-compose up` command. You will be able to view the output as the container is starting since we didn't add the `-d` flag.

---

### Useful Commands:

`docker ps` - Lists all running containers and gives you some information about the container. You don't need to be in the working folder to use this command.

![docker ps](https://raw.githubusercontent.com/jrrouse88/complete-docker-and-php/feature/chapter-3/docs-images/docker_ps.jpg)

`docker compose ps` - Lists all running services and gives you some information about the service. You need to be in the working folder (wherever the docker-compose.yaml is) in order to use this command.

![docker compose ps](https://raw.githubusercontent.com/jrrouse88/complete-docker-and-php/feature/chapter-3/docs-images/docker_compose_ps.jpg)

---

You can also use the Docker Desktop application to view any running container and get information about them.

By default, docker compose containers will be given a name of the working directory name and the service name. You can give containers names to change this. But, as you can see in the example, not specifying a name will result in `<folder-name>-<service>-<integer>`.

## Chapter Two - PHP Dockerfile

Using created images to build/run a service is one way to create a container. Another way is to have that service point to a local Dockerfile that will create an image for it. Since we want to customize our PHP installation a bit, it's the perfect time to use a Dockerfile for our PHP service.

The convention used in this tutorial will be to have all the service related things in its own folder. So we'll make a new folder called php and put the Dockerfile for our new service that uses PHP inside that new folder.

When building custom images, you have to start from a base image. You can find base images to start from in docker hub. In our case, we want a PHP image. Preferably the newest version of PHP and there are certain images with certain takeaways/add-ons that are hosted on docker hub. Most images will have a default image that may install extras and bloat. Typically, if you see `alpine` in the tag then its a very minimalist version of the image. So we'll look to use an alpine tagged version of the latest PHP version. Since we're using nginx, we also want a fpm tagged version (more on the "why" for this in chapter three).

At the time of writing, the latest version of PHP is some version of 8. So you can go to docker hub and find the php registry then search for `8.` and it should give you a list of the latest added images.

---

https://hub.docker.com/_/php/tags?page=1&ordering=last_updated&name=8.2-fpm-alpine

![docker hub php image](https://raw.githubusercontent.com/jrrouse88/complete-docker-and-php/feature/chapter-3/docs-images/php_image.jpg)

---

Since were building our own image from a base image of PHP, we'll have access to certain commands from that image such as `docker-php-ext-install`. Running this command will allow you to install php extensions which we want to do in our case since we got the alpine version of PHP. We want to add extensions to run MySQL so we install the `pdo` and `pdo_mysql` PHP extensions.

---

### Useful Commands:

You can use docker compose to build the images and run the services. However, if you wanted to build an image used for a service on its own, you can use the following command:
```shell
docker build -t jrouse:php82 -f php/Dockerfile .
```

The `-t` flag is for tagging the image and give it a name. The `-f` flag is for designating the Dockerfile location. By default, docker looks in the current working directory for the Dockerfile. Since we want to build a Dockerfile in a different location, we pass the `-f` flag and the route to the Dockerfile. Lastly, the period is the context from which you want to run the command. Since we're in our project folder, were running the command from that folder and looking into the php folder for the Dockerfile.

---

## Chapter Three - nginx Configuration

In chapter one, we setup the web service using the nginx image from docker hub and what we got was the default nginx page.

![nginx default page](https://raw.githubusercontent.com/jrrouse88/complete-docker-and-php/feature/chapter-3/docs-images/nginx_default.jpg)

We want to setup the configuration to point to wherever we will host our web files. We can take a look at the current default configuration by going into the container using the `docker exec` command.

---

### Useful Commands:

`docker exec -it docker-php_web_1 sh`

This command allows you to interact with a shell inside the container. The `sh` argument is for the shell. 

---

In order to actually see the configuration you need to know the path to the file. Normally, nginx conf files are stored in `/etc/nginx/conf.d/default.conf`. So once you start the shell for the container you can `cat` that path to view the default nginx configuration file.

![default nginx configuration](https://raw.githubusercontent.com/jrrouse88/complete-docker-and-php/feature/chapter-3/docs-images/default_nginx.jpg)

The configuration file we create here is very basic. For production, you'd normally point to an alternative configuration that has error handling and whatnot.

Nginx can't handle PHP processing directly. It relies on a separate PHP processor to handle PHP requests. That's where the fpm comes in. `php-fpm` is a processor extensively tested to work with nginx. Every key in an nginx conf file is called a directive and every directive is telling nginx to do something. You can nest directives like having a `location` directive inside the `server` directive. You can also have sibling directives, two different `server` directives for example.

Location directives use pathing to apply any child directives. For example, directives inside of `location ~ \.php$` will execute if the requested path ends in .php.

`$` prefixed things inside the conf file are variables which nginx knows the value of.

## Chapter Four - Volume Mounts

Adding a volume key to the docker compose will let docker know to use whatever file you point to as the file for the directed path in the container.

## Chapter Five - A Basic Application

Creating a basic app that will do "translations". The first attempt for the translation will pull from db and store in the cache. Subsequent lookups will use the cache instead of the db.

## Chapter Six - Adding MySQL

One way of volume mounting is to point from a local file to a container location. Another way is to use a volume name and let docker decide the best way of mounting that volume. That is how we'll handle the mysql mount. To do this you add a new key called volumes at the same level as the service key. Inside of that key you add the volume name and leave the value empty. After the empty volume has been created, you can add it as a volume to the service and point to a location in the container.

Volume key:

```docker-compose
volumes:
	mysqldata:
```

Volume mount:

```docker-compose
  db:
    image: mysql:8.0
    volumes:
      - mysqldata:/var/lib/mysql
```

