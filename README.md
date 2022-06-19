# docker-crash-course
All course files for the [Docker Crash Course tutorial on the Net Ninja site &amp; YouTube channel](https://www.youtube.com/watch?v=31ieHmcTUOk&list=PL4cUxeGkcC9hxjeEtdHFNYMtCpjNBm3h7&index=1)

You can find all codes is used in tutorial in the <b>lesson-n</b> branches.

NOTE:
Forked and added notes to readme by Konul Mammadova.
Have a nice read! 

This readme file already contains 
- all docker codes in the tutorial
- docker commands which doesn't present in the branches
- some explanation of docker topics and terminologies.


# lesson-5

Docker image is made up from layers(typically there will be parent image as the first layer).

DockerFile example:

```
FROM nodejs

WORKDIR /app

COPY . .

#running as image is creating
RUN npm install

#exposed is used for mapping our computer port to container port, actually not required when we use cmd
EXPOSE 4000

#running after building image(when we have a container)
CMD ["node", "app.js"]
```
For creating image run(-t => tag):
```
docker build -t <new_image_name> <path/to/the/dockerfile>
```


# lesson-6

We can put file and folder names to ```.dockerignore``` file for not coping to the image(COPY line) 


# lesson-7

To list docker images run the command:
```docker images```

To run an image(create container):
```docker run <image_name>```

To give a specific name to the container when running image:

```docker run --name <give_a_name_to_container> <image_name>```

To see containers list:
```docker ps``` (ps=> process)

If we don't map containers exposed port to our local computer's port, we can't acces the application.

Note for port mapping: 4005:4000 -> means localhost's 4005 is mapping to containers 4000 port

```docker run --name <new_container_name> -p 4000:4000 <image_name>``` (-p means publish)

If you want to run a container without blocking your terminal you can add detached flag:

```docker run --name <new_container_name> -p 4000:4000 ```-d``` <image_name>```


# lesson-8

## Layer Caching

When we make a change to our app, we need to create new image. When we build a new image it take less time because of layer caching(docker caches our image layers).

Every time docker tries to build an image and walks through different layers(lines), it stores that image at each layer(line) in the cache. 

So for the first time we build this image after each layer docker took our image and store it in the cache.
When we build image again before docker starts to whole build process from scratch, it looks in our cache and it tires to find an image in the cache that it can use for the new created image.

Exp: If we make a change to any file that needs also is coping into the image, it affects ```COPY``` layer of the image and therefore every further layers that is build on top of this layer(in our case: run, expose, cmd. See dockerfile example above). But since above layers(parent image and WORKDIR) aren't affected by this change, docker finds the cached images for these layers and grabs these, then runs through the subsequent layers and adds then on top to build new image. As a result the second time it takes less time to build new image.

### WHY all layers on the top of affected layer(COPY) is affected while the layers before it are not affected???

- Just because for being on top of the affected layer. In every layer(line) image is cached in the format of including whole stack of previous layers. Hence every layer on top of the affected layer needs to build from scratch.

### Optimized dockerfile example:
We can update our dockerfile example above(see lesson-5) if we need cache some further layers(layers on top of the COPY):

```
FROM nodejs

WORKDIR /app

COPY package.json .

#we changed this layer order, 
#because we don't want to npm install for every file change process and need to cache this layer.
#But we need package.json file to npm install before coping all files to the image.
#Hence we add COPY layer on top of this layer.
RUN npm install

COPY . .

#exposed is used for mapping our computer port to container port, actually not required when we use cmd
EXPOSE 4000

#running after building image(when we have a container)
CMD ["node", "app.js"]
```

# lesson - 9

## Managing images and containers

Start and stop docker:

```docker stop <container_name_or_container_id>```

```docker start <container_name_or_container_id>```

By default start runs in detached mode. If you want to run in interactive mode  use this: ```docker start -i <container>```

To remove image:

```docker image rm <image_id_or_name>```

To force removing image if there is any running container:

```docker image rm <image> -f```

To remove container:

```docker container rm <container_id_or_name>```

### Versioning images(adding tags)

To remove all images, all containers and all volumes at the same time:
```docker system prune -a```

To build an image with any tag:
```docker build -t <image_name>:v1 path/to/the/dockerfile```


# lesson-10

### Docker volumes

Ones an image is made up it becomes read only meaning that if we change the applicatio in any way, we have to rebuild a new image.
Instead of rebuild image over and over again we can use docker volumes. Volumes allow us to specify folders on our host computer that can be made available to the running container and we can map those folders on the host to the specifix folder in the container.

Note that image itself does not change. Volumes just give us a way to map directories between containers and the hos computer. So if we want to update the image to share others or create new containers, then you will have to rebuild the image.

### Implementing volume over the node project example

1. We have the dockerfile on the below:

```
FROM nodejs

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 4000

CMD ["node", "app.js"]
```

2. In the node project whenever we make any change, projects is needed to restart. ```nodemon``` is the tool for listening project folder to catch any change and automatically restart it. Therefore before using docker volume feature we update our dockerfile by adding the command to install nodemon tool globally in the project(in the container) and updating last layer to run the command ```nodemon app.js``` or just ```npm run dev``` in which dev specifies the command(nodemon app.js) in the package.json file(see package.json file):

```
FROM nodejs

RUN npm install -g nodemon

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 4000

CMD ["npm", "run", "dev"]
```
3. To build new version of image: ```docker build -t <image_name>:<version>```

4. Now we have new image that can catch any change in the container.

But we didn't map any folder to the container's specific folder to sync local changes with container yet. That's why with current image if we run and create new container, it couldn't get any change in the files.
 
To be capable of passing and syncing changes:

- We don't need to update Dockerfile(if we would do, we would have to create image again)
- We create new container on the command line: 

```docker run --name myapp:nodemon -p 4000:4000 -v <absolute_path_to_the_app_folder>:/app --rm <image_name>:<version>``` 

(```-v``` -> indicates volume, ```--rm``` -> indicates that after stopping the container it will be removed automatically)

Now we have a container which can get any change action from our local computer. We can test by changing in any file.

5. But if we remove node_modules folder by mistakenly or for another reason, it would be removed from the /app folder in the container. We just need to exclude node_modules/ changes from the volumes. To do this we create anonymous volume in the cmd:

```docker run --name myapp:nodemon -p 4000:4000 -v <absolute_path_to_the_app_folder>:/app -v /app --rm <image_name>:<version>```

(```-v /app``` -> /app in the container)

Note: It's a bit long-winded way of creating new container with these commands, but in the following lesson we will use easier way of doing this by using docker compose.


# lesson-11

