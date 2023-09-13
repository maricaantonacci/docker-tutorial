

The objective here is to try to use multi-stage images in practice. To this end we will change our Dockerfile to:

- give a nickname to the first stage: compiler

- add a second stage using the same ubuntu base image

- add the hello binary to the second stage

- make sure that CMD is in the second stage

### Mulit-stage Docker file: example

Here is the final Dockerfile:

```Dockerfile
FROM ubuntu AS compiler
RUN apt-get update
RUN apt-get install -y build-essential
ADD https://raw.githubusercontent.com/docker-library/hello-world/master/hello.c  /hello.c
RUN make hello
FROM ubuntu
COPY --from=compiler /hello /hello
CMD /hello
```
Let's build it, and check that it works correctly:
```bash
docker build -t hellomultistage .
```

and now we can test: 

```Bash
docker run hellomultistage
```

### Home work

List our images with docker images, and check the size of:

- The ubuntu base image,

- The single-stage hello image,

- The multi-stage hellomultistage image.

We can achieve even smaller images if we use smaller base images ( i.e. Apline etc ) 

