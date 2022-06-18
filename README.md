# docker-crash-course
All course files for the Docker Crash Course tutorial on the Net Ninja site &amp; YouTube channel.

# lesson-5
DockerFile example:

FROM nodejs

WORKDIR /app

COPY . .

#running as image is creating
RUN n pminstall

#exposed is used for mapping our computer port to container port, actually not required when we use cmd
EXPOSE 4000

#running after runnin image(when we have a container)
CMD ["node", "app.js"]

lesson-6

lesson-8

lesson-10

lesson-11

