# docker-crash-course
All course files for the [Docker Crash Course tutorial on the Net Ninja site &amp; YouTube channel](https://www.youtube.com/watch?v=31ieHmcTUOk&list=PL4cUxeGkcC9hxjeEtdHFNYMtCpjNBm3h7&index=1)

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

```docker run --name <new_container_name> -p 4000:4000 <image_name>``` (-p means publish)

If you want to run a container without blocking your terminal you can add detached flag:

```docker run --name <new_container_name> -p 4000:4000 ```-d``` <image_name>```

Start and stop docker:

```docker stop <container_name_or_container_id>```

```docker start <container_name_or_container_id>```

# lesson-8

# lesson-10

# lesson-11

