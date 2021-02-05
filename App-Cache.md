## What is App Cache?

App Service provides a quick way to deploy code apps and get started. During this deployment process, the code is copied to a CIFS(Network File System base) volume and the app runtime process picks this new copy of the code. This works great for many apps but we have observed issues recently for some framework runtimes that lock files and leave stale file handles on the CIFS(NFS based) volume. This causes the deployment build process unable to override that file rendering the deployments to fail. During the regular platform upgrades we also make the app CIFS volume read only which may affect availability for some apps that need access to a R/W volume. To solve this, we have built a new feature called _App Cache_. With this feature, each app instance(or VM in the App Service plan) gets it's own local ephemeral copy of the App Code instead of using the shared NFS mount which is used by default on App Service on Linux. This means if an app is running on multiple instances, each container of that app will use it's own local host(/VM) volume to host the app code instead of using the shared storage between instances. This would completely remove the dependency on the NFS volume for the app and shield it from issues in the NFS.

## How do I enable App Cache?

You need to set the app setting `WEBSITES_ENABLE_APP_CACHE=true` and you are good to go!

## Workflow

When the app setting is set, the container goes through a recycle to pick up the new app setting. This means the current container will be stopped and a new one will come up with the new app setting. The build is performed in a [build container/agent](https://github.com/Azure-App-Service/KuduLite) by a Build System called [Oryx](https://github.com/microsoft/oryx) and build artifacts are generated in a zip file. Information regarding this zip file is stored in `/home/data/SitePackages`. The App cache code reads this information, creates a docker volume per container and extracts the built artifacts into this docker volume. The docker volume is then mounted onto the `/home/site/wwwroot` of the app and the app content is served from there. The lifecycle of this docker volume is tied to the container and the docker volume will be deleted if the current container goes down.

## Things to consider

 - As the build process for app cache is different with the added overhead of extraction of built artifacts, cold start times might be slower. Once the app is up, there should be no difference in the performance of the app.
 - When the app setting `WEBSITES_ENABLE_APP_CACHE` is enabled, a new deployment is necessary as build artifacts generated when app cache is enabled are different from those when app cache is disabled.
 - Incase the NFS volume goes down, the main app will remain in a functioning state. As the SCM site depends on the NFS volume, deployments will not be possible during this time.
 - There will be no shared volumes between multiple instances of an app.
 - Currently the docker volume created is read only. So the app cannot write any file etc. onto the volume.

# How can I file an issue?
You could either open a support incident on Azure or Open an issue in [ImageBuilder Issues](https://github.com/azure-app-service/imagebuilder/issues) for any issues encountered on this regard.
<br /><br />
Thanks,<br />
App Service Linux team<br />
Microsoft.