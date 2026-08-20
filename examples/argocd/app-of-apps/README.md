# App of Apps
## Explanation
App of apps-style deployments use a single Application as a root app to install all others. Your bootstrap will typically add in a root-app.yaml and it will load all other apps from your repo, syncing them automatically as you add/remove apps.

## Benefits
App of apps allow you to have 1 standard bootstrap insertion point for your apps and load things gated by a values file. You can have your values enable/disable apps easily with this method, you don't have to manually subscribe the cluster to apps (git can manage what apps clusters deploy), and you can change the default health check LUA to make sync waves work (install wave 1, then 2... until N waves).

## Production Issues
If app A is a part of sync wave 1 and our code has an issue, any app that is after it won't be able to deploy. You *can* force your way around these issues in the Argo UI for sure, but it's manual and with Argo doing all the work for you 99% of the time it feels annoying. Example scenario:

* Bill installed Grafana, Kyverno, and Postgres apps
* Postgres has an issue where it adds a field now we aren't tracking in Git, so Argo thinks it's constantly unhealthy
* Bill tries to deploy a new Grafana dashboard via a configmap, but it hangs until Postgres gets a RCA/fix

## Files
[root-app.yaml](./root-app.yaml)
The root app that runs the "app of apps" install. This is installed once at bootstrap and then all apps load from Git automatically.
