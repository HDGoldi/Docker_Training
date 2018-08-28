Tips:

Code snippets are shown in one of three ways throughout this environment:

1. Code that looks like `this` is sample code snippets that is usually part of an explanation.
2. Code that appears in box like the one below can be clicked on and it will automatically be typed in to the appropriate terminal window:
```.term1
uname -a
```
3. Code appearing in windows like the one below is code that you should type in yourself. Usually there will be a unique ID or other bit your need to enter which we cannot supply. Items appearing in <> are the pieces you should substitute based on the instructions.
```
docker start <container ID>
```

## 1.0 Running your first container

```.term1
docker run hello-world
```

![Hello world explainer](/images/ops-basics-hello-world.svg)

## 1.1 Docker Images
```.term1
docker image pull alpine
```

You can use the `docker image` command to see a list of all images on your system.

```.term1
docker image ls
```
```
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
alpine                 latest              c51f86c28340        4 weeks ago         1.109 MB
hello-world             latest              690ed74de00f        5 months ago        960 B
```

### 1.1 docker Run

```.term1
docker run alpine ls -l
```
```
total 48
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 bin
drwxr-xr-x    5 root     root           360 Mar 18 09:47 dev
drwxr-xr-x   13 root     root          4096 Mar 18 09:47 etc
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 home
drwxr-xr-x    5 root     root          4096 Mar  2 16:20 lib
......
......
```

![docker run explainer](/images/ops-basics-run-details.svg)

```.term1
docker run alpine echo "hello from alpine"
```
And you should get the following output:
```
hello from alpine
```

```.term1
docker run alpine /bin/sh
```

Might not work?

```.term1
 docker run -it alpine /bin/sh
```

```.term1
docker ls
```
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Since no containers are running, you see a blank line. Let's try a more useful variant: `docker ls -a` 

```.term1
docker ls -a
```
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
36171a5da744        alpine              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
a6a9d46d0b2f        alpine             "echo 'hello from alp"    6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
ff0a5c3750b9        alpine             "ls -l"                   8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```

![docker instances](/images/ops-basics-instances.svg)

### 1.2 Container Isolation

```.term1
docker run -it alpine /bin/ash
```

```
 echo "hello world" > hello.txt

 ls
```
To show how isolation works, run the following:
```.term1
docker run alpine ls
```

Once again run the

```.term1
docker ls -a
```

command again and you should see output similar to the following:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
36171a5da744        alpine              "ls"                     2 minutes ago       Exited (0) 2 minutes ago                        distracted_bhaskara
3030c9c91e12        alpine              "/bin/ash"               5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
a6a9d46d0b2f        alpine             "echo 'hello from alp"    6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
ff0a5c3750b9        alpine             "ls -l"                   8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```
Graphically this is what happened on our Docker Engine:
![docker isolation](/images/ops-basics-isolation.svg)

```
docker exec <container ID> ls
```

This time we get a directory listing and it shows our "hello.txt" file because we used the container instance where we created that file.

![docker exec command](/images/ops-basics-exec.svg)


### Run a background MySQL container

Background containers are how you'll run most applications. Here's a simple example using MySQL.

1. Run a new MySQL container with the following command.

    ```.term1
    docker run \
    --detach \
    --name mydb \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    mysql:latest
    ```

2. List the running containers.

    ```.term1
    docker ls
    ```

    Notice your container is running.

    ```
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS            NAMES
    3f4e8da0caf7        mysql:latest        "docker-entrypoint..."   52 seconds ago      Up 51 seconds       3306/tcp            mydb
    ```

3. You can check what's happening in your containers by using a couple of built-in Docker commands: `docker logs` and `docker top`.

    ```.term1
    docker logs mydb
    ```

      Let's look at the processes running inside the container.

    ```.term1
      docker top mydb
    ```

    You should see the MySQL daemon (`mysqld`) is running in the container.

    ```
    PID                 USER                TIME                COMMAND
    2876                999                 0:00                mysqld
    ```


4. List the MySQL version using `docker exec`.


    ```.term1
    docker exec -it mydb \
    mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
    ```

    You will see the MySQL version number, as well as a handy warning.

    ```
    mysql: [Warning] Using a password on the command line interface can be insecure.
    mysql  Ver 14.14 Distrib 5.7.19, for Linux (x86_64) using  EditLine wrapper
    ```

### Clone the Lab's GitHub Repo & Build a simple website image

```.term1
    git clone https://github.com/dockersamples/linux_tweet_app
```

1. Make sure you're in the `linux_tweet_app` directory.

    ```.term1
    cd ~/linux_tweet_app
    ```

2. Display the contents of the Dockerfile.

    ```.term1
    cat Dockerfile
    ```

    ```
    FROM nginx:latest

    COPY index.html /usr/share/nginx/html
    COPY linux.png /usr/share/nginx/html

    EXPOSE 80 443     

    CMD ["nginx", "-g", "daemon off;"]
    ```

3. In order to make the following commands more copy/paste friendly, export an environment variable containing your DockerID (if you don't have a DockerID you can get one for free via [Docker Cloud](https://cloud.docker.com)).

    You will have to manually type this command as it requires your unique DockerID.

    `export DOCKERID=<your docker id>`

4. Echo the value of the variable back to the terminal to ensure it was stored correctly.

    ```.term1
    echo $DOCKERID
    ```

5. Use the `docker image build` command to create a new Docker image using the instructions in the Dockerfile.

    ```.term1
    docker image build --tag $DOCKERID/linux_tweet_app:1.0 .
    ```
6. Use the `docker run` command to start a new container from the image you created.

    ```.term1
    docker run \
    --detach \
    --publish 80:80 \
    --name linux_tweet_app \
    $DOCKERID/linux_tweet_app:1.0
    ```

8. Once you've accessed your website, shut it down and remove it.

    ```.term1
    docker rm --force linux_tweet_app
    ```

### Push your images to Docker Hub

1. List the images on your Docker host.

    ```.term1
    docker image ls -f reference="$DOCKERID/*"
    ```
2. Before you can push your images, you will need to log into Docker Hub.

    ```.term1
    docker login
    ```
3. Push version `1.0` of your web app using `docker image push`.

    ```.term1
    docker image push $DOCKERID/linux_tweet_app:1.0
    ```

You can browse to `https://hub.docker.com/r/<your docker id>/` and see your newly-pushed Docker images. These are public repositories, so anyone can pull the image - you don't even need a Docker ID to pull public images. Docker Hub also supports private repositories.
