# bazel-nodejs-cross-compilation

Trying to demonstrate cross compiling a simple Linux OCI image from an Arm Mac.

## Repro

Run the build with the system's toolchain:

```bash
bazel build //pkgs/foo-app
```

Load and run the tarball:

```bash
docker load --input "$(bazel cquery //pkgs/foo-app --output=files)"
docker run bazel-nodejs-cross-compilation:latest
```

This will fail because the Linux VM can't run the Darwin version of Node baked into the image:

```bash
/pkgs/foo-app/bin.runfiles/_main/pkgs/foo-app/bin_node_bin/node: line 5: /pkgs/foo-app/bin.runfiles/_main/../rules_nodejs~~node~nodejs_darwin_arm64/bin/nodejs/bin/node: cannot execute binary file: Exec format error
/pkgs/foo-app/bin.runfiles/_main/pkgs/foo-app/bin_node_bin/node: line 5: /pkgs/foo-app/bin.runfiles/_main/../rules_nodejs~~node~nodejs_darwin_arm64/bin/nodejs/bin/node: Success
```

We can demonstrate this by running the Bazel build inside of its own Docker container:

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

Attempting cross compilation fails due to a missing C++ toolchain:

```bash
bazel build //pkgs/foo-app --platforms=@rules_nodejs//nodejs:linux_arm64
```

```
WARNING: Build option --platforms has changed, discarding analysis cache (this can be expensive, see https://bazel.build/advanced/performance/iteration-speed).
ERROR: /private/var/tmp/_bazel_wburgin/cea4b2bdc5dc2e21b88af478da402372/external/bazel_tools/tools/cpp/BUILD:58:19: in cc_toolchain_alias rule @@bazel_tools//tools/cpp:current_cc_toolchain:
Traceback (most recent call last):
	File "/virtual_builtins_bzl/common/cc/cc_toolchain_alias.bzl", line 26, column 48, in _impl
	File "/virtual_builtins_bzl/common/cc/cc_helper.bzl", line 219, column 17, in _find_cpp_toolchain
Error in fail: Unable to find a CC toolchain using toolchain resolution. Target: @@bazel_tools//tools/cpp:current_cc_toolchain, Platform: @@rules_nodejs~//nodejs:linux_arm64, Exec platform: @@local_config_platform//:host
ERROR: /private/var/tmp/_bazel_wburgin/cea4b2bdc5dc2e21b88af478da402372/external/bazel_tools/tools/cpp/BUILD:58:19: Analysis of target '@@bazel_tools//tools/cpp:current_cc_toolchain' failed
ERROR: Analysis of target '//pkgs/foo-app:foo-app' failed; build aborted: Analysis failed
INFO: Elapsed time: 0.296s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
ERROR: Build did NOT complete successfully
```
