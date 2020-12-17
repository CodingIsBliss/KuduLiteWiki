# Change

Python build and runtime workflow is changing for the apps built on App Service. 

# Current build process

The app builds are carried out in a [build container/agent](https://github.com/Azure-App-Service/KuduLite) by a Build System called [Oryx](https://github.com/microsoft/oryx). This container produces artifacts which are stored on an NFS volume and later run by a Runtime container defined by the user(eg: Python 3.8).

[Oryx](https://github.com/microsoft/oryx) running in the [Build Agent](https://github.com/Azure-App-Service/KuduLite) installs the dependencies listed in requirements.txt into a virtual environment named `antenv`. This build is carried out in a local build directory. This virtual environment produced by the above build is compressed and extracted by the runtime at the root of the container to enable the users to have an optimal build and startup time.

This caused a problem where the Virtual Environment could not resolve certain app dependencies. The reason for this is that during the creation of the Virtual Environment `pip` hardcodes certain paths for dependencies. Since, the builds are carried out in an Build Container and Run on a Runtime container at a different path, this causes a mismatch between the directory structure for the virtual environment.

# Change/New Build+Runtime Process

The build workflow remains the same with respect to the use of a temporary build directory as described above. But now the whole app output(including the Virtual Environment) is compressed into a tarball and stored at `/home/site/wwwroot` with a manifest file `oryx-manifest.toml`. This manifest file  describes how the build was carried out in the build container. The runtime container on startup reads this manifest and recreates a **local** copy of the folder structure on every instance. This change also ensures that the runtime during the app startup will already have the virtual environment activated and users can run commands like `migrate` without the need to activate the environment.

As a part of this change, the app would no longer run from `/home/site/wwwroot` but would have a dynamic runtime directory. This path can be retrieved by using the variable $APP_PATH.

# Potential impact
We do not expect apps execution to break due to this change. However, for those customers who use a bash/shell script as the startupScript, there could be potential compatibility issues when the new builds are carried out after the next platform update.


Few scenarios which could break
- Apps which have hardcoded the path to their virtual environment as "/antenv/bin/activate" in their startup scripts may get an error. We recommend that these customers rebuild the app, review their startup script and modify all these paths relative to `$APP_PATH`.

- Apps which are writing to a local path relative to the app path would no longer have their data persisted. Only the data written under the `/home` directory is persisted. This can include libraries which persist session state.


# Impacted Versions:
Python 2.7, 3.6, 3.7, 3.8

# How can this be tested?
The Canary regions (Central US EUAP), West Central US, East US, West Europe regions are already upgraded with the above change. This is a good candidate region to deploy and test the app. 

Alternatively, you can download our latest images and test your app’s runtime locally. 

* Steps to test locally:
1. Pull the Build Image: `docker pull mcr.microsoft.com/appsvc/kudulite:20201109.1`
1. Create a local folder which would be used by both the build and the runtime container.
1. Start the build image locally: `docker run -p -v <local-directory>:/home <local-build-port>:8181 mcr.microsoft.com/appsvc/kudulite:20201109.1`
1. Clone the App Locally: `git clone http://localhost:<local-build-port>/localsite.git`
1. Push the app code to the build container: `git push`
1. Start the runtime container: `docker run -p <local-runtime-port>:8080 -v <local-directory>:/home mcr.microsoft.com/appsvc/python:<ver>_20201109.1`
1. Browse your app: `http://localhost:<local-build-port>/`

The latest image tags are:
* Build Container: `mcr.microsoft.com/appsvc/kudulite:20201109.1`
* Python 2.7: `mcr.microsoft.com/appsvc/python:2.7_20201109.1`
* Python 3.6: `mcr.microsoft.com/appsvc/python:3.6_20201109.1`
* Python 3.7: `mcr.microsoft.com/appsvc/python:3.7_20201109.1`
* Python 3.8: `mcr.microsoft.com/appsvc/python:3.8_20201109.1`

We are hopeful that this set of changes will ensure a better experience for our customers on App Service Linux.

# How can I file an issue?
Open an issue in [ImageBuilder Issues](https://github.com/azure-app-service/imagebuilder/issues) for any issues encountered on this regard.
<br /><br />
Thanks,<br />
App Service Linux team,<br />
Microsoft.
