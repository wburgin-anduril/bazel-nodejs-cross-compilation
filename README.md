# bazel-nodejs-cross-compilation

Trying to demonstrate cross compiling a simple Linux OCI image from an Arm Mac.

## Instructions

Run the build with the system's toolchain:

```bash
bazel build //pkgs/foo-app
```

Load and run the tarball:

```bash
docker load --input "$(bazel cquery //pkgs/foo-app --output=files)"
docker run bazel-nodejs-cross-compilation:latest
```

## Building in Docker

```bash
# This is easier if we start from a container that already has NPM installed
docker pull node:20

docker run -it -v "$(PWD):/git" --entrypoint bash --name bazel-nodejs-cross-compilation node:20
```

Re-run the build inside of the container:

```bash
/git/bazel build //pkgs/foo-app
```

Copy the tarball back to the host, load it, and run it. This should print "Hello, world":

```bash
docker cp bazel-nodejs-cross-compilation:/git/bazel-bin/pkgs/foo-app/foo-app/tarball.tar bazel-nodejs-cross-compilation.tar
docker load --input bazel-nodejs-cross-compilation.tar
docker run bazel-nodejs-cross-compilation:latest
```
