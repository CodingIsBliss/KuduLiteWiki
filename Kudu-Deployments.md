Kudu can build apps for NodsJS, Python, Dotnetcore, PHP and Ruby. It uses a build system called Oryx.

The default timeout for the deployments in kudulite is 40 mins after which the deployment will be cancelled. You can increase the timeout by setting the app setting SCM_DEPLOYMENT_INVALIDATION_MINS. Set it to a integer value. Example: SCM_DEPLOYMENT_INVALIDATION_MINS=50 
