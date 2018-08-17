# Direct Docker Access
> This is a new feature and is currently restricted to specific users. Contact the Wercker team if you would like to use it.
Sometimes your pipelines need to be able to access a Docker daemon directly. You can now use Docker commands such as docker build, docker run, docker push in your pipeline, with full access to all their features.
> Note: This is a new Note.
## Configuration
Simply set the property _docker: true_ in your pipeline and a Docker daemon will be provisioned exclusively for your pipeline. When your pipeline terminates, the daemon (and the host on which it runs) is destroyed.
build:  
 box: alpine  
 docker: true  
 steps:  
 ...
You can use the _docker_ command or any tool or a library that requires direct access to a Docker daemon. The environment variable _DOCKER_HOST_ is set to the URI of the daemon.
Note that if your pipeline needs to use the _docker_ command, you will need to specify a box image that already has Docker installed or add a step to install it.
## Environment Variables
The following additional environment variables are made available in your pipeline:
|Variable Name | Example | Description |
|--------------|---------|-------------|
|_DOCKER_HOST_| _w-HuMwpXsKhN_ |URI of docker daemon|
|_DOCKER_NETWORK_NAME_| _tcp://10.15.54.245:14567_| Name of Docker bridge network used by the box (pipeline container) and the services|
## Docker Networks
The pipeline container (and any services that you specify) are connected to a Docker bridge network created by Wercker. The name of the network is available in the environment variable _$DOCKER_NETWORK_NAME_.
This means that if your pipeline starts a container, then you should connect it to the same network. This allows the pipeline container (and other containers on that network) to create connections to the new container using the name of the container as the hostname.
For example, if you want to use the _docker run_ command to start a container, then you can do this by adding the _--network=$DOCKER_NETWORK_NAME_ parameter.
## Example
The following example shows a pipeline which requires direct access to a Docker daemon, and therefore specifies _docker: true_ property.
The pipeline starts by installing _docker_ and _curl_.
It then uses the _docker run_ command to start a container using a sample image from Docker that starts a simple web server. The _--network=$DOCKER_NETWORK_NAME_ parameter is used to add the container to the same network as the pipeline container.
The _curl_ command is then used to send a HTTP **GET** request to the web server. Note that the name of the container is used as the hostname, which is only possible because we specified _--network=$DOCKER_NETWORK_NAME_.
Finally, the _docker kill_ command is used to kill the container.
## How does direct Docker access relate to the internal steps for Docker?
Wercker already has some built-in steps to perform basic Docker operations. These are _internal/docker-build_, _internal/docker-push_, _internal/docker-scratch-push_, _internal/docker-run_ and _internal/docker-kill_. For more information see [Internal Steps] and [Building an image].
[Internal Steps]: https://devcenter.wercker.com/development/steps/internal-steps/
[Building an Image]: https://devcenter.wercker.com/administration/containers/building-an-image/
## Using the Workflow Editor to create fan-in connections
Fan-in connections come in handy when you want a particular pipeline to depend on specific preceding pipeline(s) to complete executing and also select the source of the artifact it is going to consume.
In the following example, the _system-test_ pipeline is waiting for the _unit-test_ and _integration-test_ pipelines to finish, and is also going to consume artifacts from both the pipelines. ![Workflow](https://devcenter.wercker.com/development/workflows/managing-pipeline-artifacts/workflow-editor-for-fan-in.png)
To establish a fan-in connection using the Workflow Editor:
1. Click a pipeline on which the successive pipeline is depending. For example, **integration-test**.
2. Click the pipeline that you want to fan into. For example, **system-test**.
By default, only the artifact of the pipeline that originally preceded is made available to the pipeline that now has a fan-in connection. If you want the artifacts of additional or all the previous pipelines to be made available - or remove any fan-in sources, click the **edit** icon on the pipeline you that are currently configuring. The **Manage source pipeline** dialog box is displayed.
