## Idea

The idea is for each app instance to have its own storage based on the local storage of the VM instead of using the shared NFS mount which is used by default on App Service Linux. This means if an app is on multiple instances, each container of that app will use local storage on the VM it is running on instead of using the shared storage between instances. This should completely remove the dependency on the NFS volume for the app and shield it from issues in the NFS.

## How do I enable App Cache?

You need to set the app setting `WEBSITES_ENABLE_APP_CACHE=true` and you are good to go!

## App Cache Process

When the app setting is set, the container goes through a recycle to pick up the new app setting. This means the current container will be stopped and a new one will come up with the new app setting. The build is performed in a [build container/agent](https://github.com/Azure-App-Service/KuduLite) by a Build System called [Oryx](https://github.com/microsoft/oryx) and build artifacts are generated in a zip file. Information regarding this zip file is stored in `/home/data/SitePackages`. The App cache code reads this information, creates a docker volume per container and extracts the built artifacts into this docker volume. The docker volume is then mounted onto the `/home/site/wwwroot` of the app and the app content is served from there. The lifecycle of this docker volume is tied to the container and the docker volume will be deleted if the current container goes down.

## Things to consider

 - As the build process for app cache is different with the added overhead of extraction of built artifacts, cold start times might be slower. Once the app is up, there should be no difference in the performance of the app.
 - When the app setting `WEBSITES_ENABLE_APP_CACHE` is enabled, a new deployment is necessary as build artifacts generated when app cache is enabled are different from those when app cache is disabled.
 - Incase the NFS volume goes down, the main app will remain in a functioning state. As the SCM site depends on the NFS volume, deployments will not be possible during this time.
 - There will be no shared volumes between multiple instances of an app.
 - Currently the docker volume created is read only. So the app cannot write any file etc. onto the volume.