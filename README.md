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

Start and stop docker:

```docker stop <container_name_or_container_id>```

```docker start <container_name_or_container_id>```


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


# lesson-10


# lesson-11

