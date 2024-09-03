This section is about another Dockerfile keyword: **COPY**.

During the previous sections we have installed things in our container images by downloading packages. In the real life we might also do something slightly different such as: copy files from the build context to the container that we are building.

!!! tip 
    Remember: the build context is the directory containing the Dockerfile.

### Build some C code

In the following simple example we want to build a container that compiles a basic "Hello world" program in C.

Example, a hello.c:

```c
#include <stdio.h>

int main () {
  puts("Hello, world!");
  return 0;
}
```
We can create a new directory, and put this file in there.

Then we will write the Dockerfile and we will use COPY to place the source file into the container

!!! tip
    On Debian and Ubuntu, the package build-essential will get us a compiler.
    When installing it, don't forget to specify the -y flag, otherwise the build will fail (since the build cannot be interactive).


```Dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y build-essential
COPY hello.c /
RUN make hello
CMD /hello
```

**Exercise:**

- Create hello.c and Dockerfile in the same directory:
- Run docker build -t hello . in this directory.
- Run docker run hello, you should see Hello, world!.

### COPY and the build cache

Docker can cache steps involving COPY. Those steps will not be executed again if the files haven't been changed. You can try it yourself by
- Run the build again, but now modify hello.c 

### The .dockerignore file

Something you need to take care of is to avoid copy of unneeded files i.e. files in your context but not required in the image. To do that you have a handle: **.dockerignore**

You can create it at the top-level of the build context specifying file names and globs to ignore

They won't be sent to the builder and won't end up in the resulting image

See the [documentation](https://docs.docker.com/engine/reference/builder/#dockerignore-file) for the little details



