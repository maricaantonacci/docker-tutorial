An environment variable consists of a variable name and its value.

There are two ways to set environment variables for a docker container: with CLI arguments, using an _env_ file.

### CLI arguments

When we launch our Docker container, **we can pass environment variables as key-value pairs directly into the command line** using the parameter _–env_ (or its short form _-e_).

For instance, let's execute the following command:

```
docker run --rm --env VARIABLE1=foobar alpine env
```

!!! tip
    Docker Alpine is the “Dockerized” version of Alpine Linux, a Linux distribution known for being exceptionally lightweight and secure. For these reasons and others, Docker Alpine is a popular choice for developers looking for a base image on which to create their own containerized apps.

The environment variables we set will be printed to the console:

```
...
VARIABLE1=foobar
```

As can be seen, the Docker container correctly interprets the variable `VARIABLE1`.

Also, **we can omit the value in the command line if the variable already exists in the local environment**.

For example, let's define a local environment variable:

```
export VARIABLE2=foobar2
```

Then, let's specify the environment variable without its value:

```
docker run --rm --env VARIABLE2 alpine env
```

And we can see Docker still picked up the value, this time from the surrounding environment:

```
...
VARIABLE2=foobar2
```

### Using --env-file

The above solution is adequate when the number of variables is low. However, as soon as we have more than a handful of variables, it can quickly become cumbersome and error-prone.

**An alternative solution is to use a text file to store our variables**, using the standard _key=value_ format.

Let's define a few variables in a file we'll call _my-env.txt_:


```
echo VARIABLE1=foobar1 > my-env.txt
echo VARIABLE2=foobar2 >> my-env.txt
echo VARIABLE3=foobar3 >> my-env.txt
```

Now, let's inject this file into our Docker container:

```
docker run --env-file my-env.txt alpine env
```

Finally, let's take a look at the output:

```
...
VARIABLE1=foobar1
VARIABLE2=foobar2
VARIABLE3=foobar3
```
