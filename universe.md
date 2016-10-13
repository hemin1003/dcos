Universe

### **Submit your Package**

Developers are invited to publish a package containing their DC\/OS Service by submitting a Pull Request targeted at the `version-3.x` branch of this repo.

Full Instructions:

1. Fork this repo and clone the fork:

  ```
  git clone https://github.com/<user>/universe.git /path/to/universe
  ```

2. Ensure that the `jsonschema` command line tool is installed:

  ```
  sudo pip install jsonschema
  ```

3. Run the verification and build script:

  ```
  scripts/build.sh
  ```

4. Verify all build steps completed successfully

5. Submit a pull request against the `version-3.x` branch with your changes. Every pull request opened will have a set of automated verifications run against it. These automated verification are reported against the pull request using the GitHub status API. All verifications must pass in order for a pull request to be eligible for merge.

6. Respond to manual review feedback provided by the DC\/OS Community.

  * Each Pull Request to Universe will also be manually reviewed by a member of the DC\/OS Community. To ensure your package is able to be made available to users as quickly as possible be sure to respond to the feedback provided.


## **Repository Consumption**

In order for Universe to be consumed by DC\/OS the build process needs to be run to create the Universe Server.

### **Universe Server**

Universe Server is a new component introduce alongside `packagingVersion` `3.0`. In order for Universe to be able to provide packages for many versions of DC\/OS at the same time, it is necessary for a server to be responsible for serving the correct set of packages to a cluster based on the cluster's version.

All Pull Requests opened for Universe and the `version-3.x` branch will have their Docker image built and published to the DockerHub image [`mesosphere/universe-server`](https://hub.docker.com/r/mesosphere/universe-server/). In the artifacts tab of the build results you can find`docker/server/marathon.json` which can be used to run the Universe Server for testing in your DC\/OS cluster. For each Pull Request, click the details link of the "Universe Server Docker image" status report to view the build results.

#### **Build Universe Server locally**

1. Validate and build the Universe artifacts

  ```
  scripts/build.sh
  ```

2. Build the Universe Server Docker image

  ```
  DOCKER_TAG="my-package" docker/server/build.bash
  ```

  This will create a Docker image `universe-server:my-package` and `docker/server/target/marathon.json` on your local machine

3. If you would like to publish the built Docker image, run

  ```
  DOCKER_TAG="my-package" docker/server/build.bash publish
  ```


#### **Run Universe Server**

Using the `marathon.json` that is created when building Universe Server we can run a Universe Server in our DC\/OS Cluster which can then be used to install packages.

Run the following commands to configure DC\/OS to use the custom Universe Server \(DC\/OS 1.8+\):

```
dcos marathon app add marathon.json
dcos package repo add --index=0 dev-universe http://universe.marathon.mesos:8085/repo
```

For DC\/OS 1.7, a different URL must be used:

```
dcos marathon app add marathon.json
dcos package repo add --index=0 dev-universe http://universe.marathon.mesos:8085/repo-1.7
```

