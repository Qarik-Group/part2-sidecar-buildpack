# Part 2 - Supply buildpack for Cloud Foundry sidecars

Cloud Foundry sidecars are an additional process running inside your application container (see [blog post](https://www.cloudfoundry.org/blog/how-to-push-an-app-to-cloud-foundry-with-sidecars/)). Cloud Foundry buildpacks allow the installation of additional software within your application container.

In this sample project, we use a buildpack to create a dummy executable `config-server`, and the application runs it as a sidecar.

Run through the demonstration below, and then see the highlights of parts of this repo/buildpack.

## Requirements

This demonstration of sidecars requires a Cloud Foundry running [capi-release](https://github.com/cloudfoundry/capi-release) [`1.79.0`](https://github.com/cloudfoundry/capi-release/releases/tag/1.79.0) or greater (for example [cf-deployment v7.11.0](https://github.com/cloudfoundry/cf-deployment/releases/tag/v7.11.0) or higher).

At time of writing, unfortunately Cloud Foundry running locally with [CFDev](https://github.com/cloudfoundry-incubator/cfdev) (`cf dev start`) does not yet have Sidecar support.

Fortunately, we can fix our CFDev with [this script](https://gist.github.com/drnic/777e344bb34e32e102b67f2eb2e79bc1). Hurray!

```plain
cf dev start
eval "$(cf dev bosh env)"
curl -sS https://gist.githubusercontent.com/drnic/777e344bb34e32e102b67f2eb2e79bc1/raw/344696301f133622044082fd321a75093d123951/update-cf-capi.sh | bash -
```

## Demonstration

```plain
cf v3-create-app app-using-config-server
cf v3-apply-manifest -f fixtures/rubyapp/manifest.yml
cf v3-push app-using-config-server -p fixtures/rubyapp
```

If you view the logs you'll see the sidecar's output and the ruby app's output:

```plain
$ cf logs app-using-config-server --recent
...
[APP/PROC/WEB/SIDECAR/CONFIG-SERVER/0] OUT Starting dummy config-server...
[APP/PROC/WEB/0] ERR [2019-05-18 02:53:35] INFO  WEBrick 1.3.1
[APP/PROC/WEB/0] ERR [2019-05-18 02:53:35] INFO  ruby 2.4.6 (2019-04-01) [x86_64-linux]
[APP/PROC/WEB/0] ERR [2019-05-18 02:53:35] INFO  WEBrick::HTTPServer#start: pid=16 port=8080
```

## Highlights

This project is first-and-foremostly a supply buildpack, which also includes an application for dev/testing/demonstration.

A supply buildpack can be used in addition to a normal buildpack to inject additional software/libraries/executables/files into the application droplet and runtime containers.

A supply buildpack needs:

* a [`bin/supply`](bin/supply) file
* to be included in an application's [`manifest.yml`](fixtures/rubyapp/manifest.yml) list of `buildpacks`, but cannot be the last in that list.

Our sample application's `manifest.yml` specifies this buildpack as the first in the list. It references the buildpack by its HTTPS URI to the git repository. It could have also referenced an HTTPS URI to a `.zip` file, or the name of a pre-uploaded buildpack (as found in `cf buildpacks` list).

```yaml
  buildpacks:
  - https://github.com/starkandwayne/part2-sidecar-buildpack
  - ruby_buildpack
```

The `bin/supply` can create/install files into a specific folder. This folder is provided as the first argument when `bin/supply` is executed during staging.

In our `bin/supply` script we stored this first argument in the `$BUILD_DIR` environment variable; which is a more meaningful name than `$1`.

```bash
BUILD_DIR=$1
```

Our silly demonstration buildpack created a silly demonstration script `config-server`, and set it as an executable (`chmod +x`). Our application cannot interact with this silly sidecar; because its just silliness.

In our next sample buildpack we will package up some real software that our application can interact with.

## Thanks

The sample app in `fixtures/rubyapp` and its example of running a fictional `config-server` sidecar originate from https://github.com/cloudfoundry-samples/capi-sidecar-samples/tree/master/sidecar-dependent-app.
