# Change

Python build and runtime workflow is changing for the apps built on App Service. 

# Current build process

The app builds are carried out in a [build container/agent](https://github.com/Azure-App-Service/KuduLite) by a Build System called [Oryx](https://github.com/microsoft/oryx). This container produces artifacts which are stored on an NFS volume and later run by a Runtime container for the user defined Runtime(eg: Python 3.8).

Oryx running in the Build Agent carries out the build in a local build directory where it installs the dependencies listed in requirements.txt into a virtual environment named `antenv`. This virtual environment is compressed and extracted by the runtime at the root of the container to enable the users to have an optimal build and startup time.

This caused a problem with Virtual Environment not listing certain app dependencies. The reason for this is that during the creation of the Virtual Environment `pip` hardcodes certain paths for dependencies. Since, the builds are carried out in an Build Container and run on a Runtime container at a different path, this caused a mismatch between the directory structure for the venv.

# Change/New Build+Runtime Process

The build process remains same with respect to the use of build directory for the builds but now the whole app is compressed into a tarball and stored at `/home/site/wwwroot` with a manifest file `oryx-manifest.toml`. This manifest describes how the build was carried out in the build container. The runtime container on startup reads this manifest and recreates a **local** copy of the folder structure on every instance. This change also ensures scripts during the app startup will already have the virtual environment activated and can run commands like `migrate` without the need to activate the environment.

As a part of this change, the app would no longer run from `/home/site/wwwroot` but would have a dynamic runtime directory. This path can be retrieved by using the variable $APP_PATH.

# Potential impact
We do not expect apps execution to break due to this change. However, for those customers who use a bash/shell script as the startupScript, there could be potential compatibility issues after the new builds are carried out.

One such example is for the customers who have hardcoded the path to their virtual environment as "/antenv/bin/activate" in their scripts may get an error. We recommend that these customers review their startup script and modify all these paths relative to `$APP_PATH`.

Customers whose apps are writing to a local path relative to the app would no longer have their data persisted. Only the data written under the `/home` directory is persisted.

# Impacted Versions:
Python 2.7, 3.5, 3.6, 3.7, 3.8


# How can this be tested?
The <> region is upgraded with the above change. This is a good candidate region to deploy and test the app. Alternatively, you can download our latest images and test your app’s runtime locally. The latest image tags are
Build Container:
Python 2.7: mcr.microsoft.com/appsvc/python/1.7_20201110.1
Python 3.8.

We are hopeful that this set of changes will ensure a better experience for our customers on App Service Linux.

# How can I file an issue?
Open an issue in [ImageBuilder Issues](https://github.com/azure-app-service/imagebuilder/issues) for any issues encountered on this regard.

Thanks,
App Service Linux team,
Microsoft.
