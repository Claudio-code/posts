# How to get your Spring Boot API to run with docker

The objective is help you put on api wrote using spring boot on docker, show how make build with **maven** and **gradle** because build change depending on your dependency manager favorite.
The main motivation to write it is when I was learning about java ecosystem I not found content easy to understand how put spring api in docker images to put in cloud. In two forms not change who we up containers with docker-compose.

One important thing is you never should compile project in your computer language-independent, because if for example are using a computer with one processor ARM and when it's application is running in server with x86 processor? To avoid this risk, the build is done in the container.
Even if the differences in hardware not big often small differences between hardware can make your project not run in server if you build it in another machine.

## Create Dockerfile to make build with maven

**1.** The first thing to make is construct command sequence who will do the build our loved .jar and for that we go get one image of **docker** with **maven** and **jdk** version with we need,
we get last jdk lts (17 version) and for that we will use it command and let's give it surname to it stage:

```dockerfile
FROM maven:3-openjdk-17-slim as builder
```

Let's pass to directory where will be moved files needed to build it project and let's copy all files to inside docker stage.

```dockerfile
WORKDIR /builder
COPY . .
```

Now let's pass command to generate compiled file of our api.

```dockerfile
RUN mvn clean package -DskipTests --batch-mode
```

