In this section you'll learn how to leverage CI/CD pipilines in INFN baltig (or any other gitlab instance) in order to automatically build your images and to push them to the gitlab container registry.  

## Requirements

- baltig.infn.it account and access
- gitlab CLI:
    ````bash
    wget https://gitlab.com/gitlab-org/cli/-/releases/v1.32.0/downloads/glab_1.32.0_Linux_x86_64.deb
    sudo dpkg -i glab_1.32.0_Linux_x86_64.deb
    ````
- a baltig token with 'api' and 'write_repository' scopes (see below)
- having followed the [previous](../dockerfile/exercise.md) tutorial

## Create a git repository with your app

Let's initialize a git repository in our `flask` dir:

```bash
cd flask
git init
git switch -c main
git config --global user.name "Diego Ciangottini"
git config --global user.email "diego.ciangottini@pg.infn.it"
```

Login in baltig via:

```bash
$ glab auth login
? What GitLab instance do you want to log into? GitLab Self-hosted Instance
? GitLab hostname: baltig.infn.it
? API hostname: baltig.infn.it
- Logging into baltig.infn.it
? How would you like to login? Token

Tip: you can generate a Personal Access Token here https://baltig.infn.it/-/profile/personal_access_tokens
The minimum required scopes are 'api' and 'write_repository'.
? Paste your authentication token: ********************
? Choose default git protocol HTTPS
? Authenticate Git with your GitLab credentials? Yes
? Choose host API protocol HTTPS
- glab config set -h baltig.infn.it git_protocol https
✓ Configured git protocol
- glab config set -h baltig.infn.it api_protocol https
✓ Configured API protocol
✓ Logged in as project_4873_bot
```

## Create a pipeline to automatically build your image

Create a file `.gitlab-ci.yml` with your workflow description:

```yaml
build_base_image:
  image: docker:latest
  stage: build
  services:
    - docker:dind
  before_script:
    - echo 'docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} baltig.infn.it:4567'
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} baltig.infn.it:4567
  # Default branch leaves tag empty (= latest tag)
  # All other branches are tagged with the escaped branch name (commit ref slug)
  script:
    - |
      if [[ "$CI_COMMIT_BRANCH" == "$CI_DEFAULT_BRANCH" ]]; then
        tag=":latest"
        echo "Running on default branch '$CI_DEFAULT_BRANCH': tag = 'latest'"
      else
        tag=":$CI_COMMIT_REF_SLUG"
        echo "Running on branch '$CI_COMMIT_BRANCH': tag = $tag"
      fi
      docker build -t "$CI_REGISTRY_IMAGE${tag}" .
      docker push "$CI_REGISTRY_IMAGE${tag}"
```

And then commit everything into gitlab that you already created in previous tutorials:

```bash
git remote add origin https://baltig.infn.it/ciangottini/tutorial-ci.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

!!! warning
    Remember to replace `ciangottini` with **your baltig username**!!

## Monitor the building process

Pipeline monitoring page is available at your repo site like `https://baltig.infn.it/ciangottini/tutorial-ci/-/pipelines`

Or via CLI:

```bash
$ glab ci list
Showing 3 pipelines on ciangottini/tutorial-ci (Page 1)

(running) • #82282  main  (less than a minute ago)
(failed) • #82281   main  (about 4 minutes ago)   
(failed) • #82280   main  (about 7 minutes ago)
```

and then you can get details via `glab ci view #82281`. Or you than live tracing the progress log via `glab ci trace #82285`

Once completed you should be able to see the docker image stored on gitlab registry at something like `https://baltig.infn.it/ciangottini/tutorial-ci/container_registry`

The container is now ready to be pulled from the registry via: `docker pull baltig.infn.it:4567/ciangottini/tutorial-ci`

## Dagger.io preview

- Install the [CLI](https://docs.dagger.io/quickstart/729236/cli#install-the-dagger-cli) and the [SDK](https://docs.dagger.io/quickstart/628381/sdk) via pip3, as they are needed to test the pipeline locally.
- Then create a file in `ci/main.py` with the example code doing the following:
    - build the flask image
    - instantiate a webserver for tests
    - test the response correctness
    - publish the image
 
```python
import random
import sys

import anyio
import httpx

import json

import anyio
import dagger


async def main():
    config = dagger.Config(log_output=sys.stdout)

    async with dagger.Connection(config) as client:
        # set build context
        context_dir = client.host().directory(".")

        # build using Dockerfile
        container = (
            await context_dir.docker_build()
        )

        # execute the container as a service to be tested against
        http_srv = (
            container.with_env_variable(name="ENVIRONMENT", value="dagger-test")
                .with_env_variable(name="PORT", value="8080")
                .with_exposed_port(8080)
                .as_service()
        )

        # expose HTTP service to host
        tunnel = await client.host().tunnel(http_srv).start()

        # get HTTP service address
        endpoint = await tunnel.endpoint()

        # access HTTP service from host
        async with httpx.AsyncClient() as http:
            r = await http.get(f"http://{endpoint}")
            print(r.status_code)
            print(r.text)

            result = json.loads(r.text)
            expected = {"env":"dagger-test"}
            if r.status_code==200 and result==expected:
                pass
            elif result==expected:
                print(f"{result}!={expected}")
                sys.exit(1)
            else:
                print(f"Status code: {r.status_code}")
                sys.exit(1)

        image_ref = await container.publish(f"baltig.infn.it:4567/ciangottini/tutorial-ci-23/hello-dagger-{random.randint(0, 10000000)}")

    print(f"Published image to: {image_ref}")



anyio.run(main)
```

You can easily try locally the whole chain with:

```bash
dagger run python ci/main.py
```

And just as easily integrate it into a gitlab pipeline with the following content to be appended into `.gitlab-ci.yml`:

```yaml
.docker:
  image: python:3-alpine
  services:
    - docker:dind
.dagger:
  extends: [.docker]
  before_script:
    - apk add docker-cli curl
    - cd /usr/local && { curl -L https://dl.dagger.io/dagger/install.sh | sh; cd -; }
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} baltig.infn.it:4567
build-deploy:
  extends: [.dagger]
  script:
    - apk add python3-dev gcc libc-dev
    - pip install --upgrade pip && pip install dagger-io httpx
    - dagger run python ci/main.py
```

For more details and examples visit the [Dagger.io](https://docs.dagger.io/quickstart/593914/hello) documentation
