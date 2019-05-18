# Sample supple buildpack for Cloud Foundry sidecars

## Example app

```plain
cd fixtures/rubyapp
cf v3-create-app app-using-config-server
cf v3-apply-manifest -f manifest.yml
cf v3-push app-using-config-server
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

## Thanks

The sample app in `fixtures/rubyapp` and its example of running a fictional `config-server` sidecar originate from https://github.com/cloudfoundry-samples/capi-sidecar-samples/tree/master/sidecar-dependent-app.
