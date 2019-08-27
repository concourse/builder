# builder (DEPRECATED)

This task has been deprecated in favor of the [`oci-build`
task](https://github.com/vito/oci-build-task).

For a comparison, see [differences from `builder`
task](https://github.com/vito/oci-build-task#differences-from-builder-task).

The original docs are preserved below in case you still need to reference them.

---

A Concourse task to builds Docker images without pushing and without spinning
up a Docker daemon. Currently uses [`img`](http://github.com/genuinetools/img)
for the building and saving.

This repository describes an image which should be used to run a task similar
to `example.yml`.

A stretch goal of this is to support running without `privileged: true`, though
it currently still requires it.


## task config

Concourse doesn't yet have easily distributable task configs (there's an [open
call for an RFC](https://github.com/concourse/rfcs/issues/7)), so we'll just
have to document what you need to set here.

### `image`

The task's image should refer to `concourse/builder-task`. You can either
configure this via `image_resource` or pull it in as part of your pipeline.

### `params`

The following params are required:

* `$REPOSITORY`: the repository to name the image, e.g.
  `concourse/builder-task`.

The following are optional:

* `$TAG` (default `latest`): the tag to apply to the image.

* `$TAG_FILE` (default empty): the tag should be a path to a file containing the name of the tag.

* `$CONTEXT` (default `.`): the path to the directory to build. This should
  refer to one of the inputs.

* `$DOCKERFILE` (default `$CONTEXT/Dockerfile`): the path to the `Dockerfile`
  to build.

* `$BUILD_ARG_*` (default empty): Params that start with `BUILD_ARG_` will be
  translated to `--build-arg` options. For example `BUILD_ARG_foo=bar`, will become
  `--build-arg foo=bar`

* `$BUILD_ARGS_FILE` (default empty): path to a file containing Docker build-time variables.

  Example file contents:
  ```
  EMAIL=me@yopmail.com
  HOW_MANY_THINGS=1
  DO_THING=false
  ```

* `$TARGET` (default empty): the target build stage to build.

* `$TARGET_FILE` (default empty): the path to a file containing the name of the target build stage to build.

### `inputs`

There are no required inputs - your task should just list each artifact it
needs as an input.

### `outputs`

Your task may configure an output called `image`. The saved image tarball will
be written to `image.tar` within the output. This tarball can be passed along
to `docker load`, or uploaded to a registry using the [Registry Image
resource](https://github.com/concourse/registry-image-resource#out-push-an-image-up-to-the-registry-under-the-given-tags).

Your task may configure an output called `rootfs`. A `metadata.json` and `rootfs` subfolder will
be created in the output. This can be used to start the image in a subsequent task without
uploading it to a registry using the ["image" task step option](https://concourse-ci.org/task-step.html#task-step-image).

### `caches`

Build caching can be enabled by configuring a cache named `cache` on the task.

### `run`

Your task should execute the `build` script.


## example

This repo contains an `example.yml`, which builds the image for the builder
itself:

```sh
fly -t dev execute -c example.yml -o image=. -p
docker load -i image.tar
```
