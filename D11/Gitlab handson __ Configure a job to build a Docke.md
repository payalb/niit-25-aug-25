<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Gitlab handson :: Configure a job to build a Docker image

Provide detailed step by step instruction

Here are detailed, step-by-step instructions to configure a GitLab CI/CD job that builds a Docker image using a GitLab pipeline. These steps cover preparing your codebase, creating the required GitLab configuration, and successfully building (and optionally pushing) your Docker image.

## Step 1: Prerequisites

- **You must have:**
    - A project hosted on GitLab with a valid `Dockerfile` at the repository root (or specify the path in the build command).
    - A GitLab Runner registered with **Docker-in-Docker (DinD)** support (this is necessary for running Docker commands inside the CI job).


## Step 2: Create/Edit `.gitlab-ci.yml`

At the root of your project, create (or edit) the `.gitlab-ci.yml` file. Add a job to build your Docker image.

### Minimal Example

```yaml
stages:
  - build

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  image: docker:25.0
  services:
    - docker:25.0-dind
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG         # Optional: push image to GitLab registry
```

**Breakdown of Key Elements:**

- `image: docker:25.0` – uses the official Docker image to execute build commands
- `services: - docker:25.0-dind` – enables Docker-in-Docker so Docker CLI can connect to the Docker daemon
- `script:` contains:
    - Login to GitLab's Container Registry with predefined environment variables (secure, handled by GitLab)
    - Build the Docker image and tag it (`$IMAGE_TAG`)
    - (Optional) Push the built image to your project’s Container Registry


## Step 3: Push Changes and Run the Pipeline

1. Commit your `.gitlab-ci.yml` and any changes:

```sh
git add .gitlab-ci.yml
git commit -m "Add CI job to build Docker image"
git push
```

2. On GitLab, go to **CI/CD > Pipelines**. You should see the pipeline running.
3. Click into the pipeline and watch logs from the `build` job.

## Step 4: Verify the Built Image (Optional: Push \& Use)

- If you have included the `docker push` step, after a successful pipeline you can view and pull your image from the **Packages \& Registries > Container Registry** section in the GitLab UI.
- You can use `$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA` as the tag to pull/run the image elsewhere.


## Step 5: Tips \& Best Practices

- Use distinct tags (like commit SHA) to avoid collisions and caching issues.
- Do **not** use the `latest` tag for pipeline builds; it can cause race conditions[^1].
- Add other pipeline jobs (test, deploy) as needed.
- Optionally, use `rules:` to only build images for certain branches or tags.


## Step 6: Example for Push-Only on Main Branch

```yaml
build:
  stage: build
  image: docker:25.0
  services:
    - docker:25.0-dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

This only builds and pushes images from the `main` branch[^2][^3][^4].

By following these steps, your GitLab CI/CD pipeline will reliably build Docker images directly from your repository!

<div style="text-align: center">⁂</div>

[^1]: https://blog.callr.tech/building-docker-images-with-gitlab-ci-best-practices/

[^2]: https://docs.gitlab.com/user/packages/container_registry/build_and_push_images/

[^3]: https://spacelift.io/blog/docker-ci-cd

[^4]: https://spacelift.io/blog/gitlab-ci-yml

[^5]: https://docs.gitlab.com/ci/docker/using_docker_build/

[^6]: https://www.youtube.com/watch?v=7I6tHw68DMQ

[^7]: https://docs.gitlab.com/ci/docker/using_docker_images/

[^8]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-continuous-deployment-pipeline-with-gitlab-on-ubuntu

[^9]: https://snorre.io/blog/2018-02-14-building-docker-images-with-gitlab-ci/

[^10]: https://www.reddit.com/r/gitlab/comments/17n02bd/gitlab_pipeline_and_building_docker_images/

[^11]: https://faun.pub/building-a-docker-image-with-gitlab-ci-and-net-core-8f59681a86c4

[^12]: https://docs.gitlab.com/ci/examples/

[^13]: https://docs.gitlab.com/ci/quick_start/

[^14]: https://www.reddit.com/r/docker/comments/lx9k0g/docker_build_with_gitlab_cicd/

[^15]: https://www.youtube.com/watch?v=HCuBdbuJdTU

[^16]: https://www.youtube.com/watch?v=noGcDULs9EA

[^17]: https://www.youtube.com/watch?v=AR29V1wWjjk

[^18]: https://www.youtube.com/watch?v=0M9qGD59wMQ

[^19]: https://www.youtube.com/watch?v=ZYI2mnoYW8Q

[^20]: https://www.youtube.com/watch?v=GdY_haXvgXw

[^21]: https://www.youtube.com/watch?v=FLed6nXrqfM

