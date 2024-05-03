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
