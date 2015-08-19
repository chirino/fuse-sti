# fuse-sti

An image that can be used with Openshift's [Source To Image](https://docs.openshift.com/enterprise/3.0/creating_images/s2i.html) in order to build
[Karaf4 custom assembly](https://karaf.apache.org/manual/latest/developers-guide/custom-distribution.html) or
[Hawt-app](https://github.com/jboss-fuse/hawt-app) based maven projects.

## Usage:

Using sti command:

	sti build <git repo url> dhirajsb/fuse-sti <target image name>
	docker run <target image name>

Using oc command:

    oc new-app --strategy=source dhirajsb/fuse-sti~<git repo url>

## Configuring the Karaf4 or hawt-app assembly

The location of the Karaf4 or hawt-app assembly built by the maven project can be provided in multiple ways.

- Default assembly file `*.tar.gz` in output directory, which is `target` by default.
- By using the `-e` flag in sti or oc command (e.g. `sti build -e "FUSE_ASSEMBLY=my-artifactId-1.0-SNAPSHOT.tar.gz"` ....).
- By setting `FUSE_ASSEMBLY` property in .sti/environment under the projects source.

## Customizing the build

It may be possible that the maven build needs to be customized. For example:

- To invoke custom goals.
- To skip tests.
- To provide custom configuration to the build.
- To build specific modules inside a multimodule project.
- To add debug level logging to the Maven build.

The `MAVEN_ARGS` environment variable can be set to change the behaviour. By default `MAVEN_ARGS` is set as follows.

Karaf4:

    install karaf:assembly karaf:archive -DskipTests -e

Hawt-App:

    package hawt-app:build -DskipTests -e

You can override the `MAVEN_ARGS` like in the example below we tell maven to just build the project with groupId "some.groupId" and artifactId "some.artifactId" and all its module dependencies.

	sti build -e "MAVEN_ARGS=install -pl some.groupId:some.artifactId -am" <git repo url> dhirajsb/fuse-sti <target image name>

You can also just override the `MAVEN_DEBUG_ARGS` environment variable with:

    -e "MAVEN_DEBUG_ARGS=-X"

## Working with multimodule projects
The example above is pretty handy for multimodule projects. Another useful option is the OUTPUT_DIR environment variable. This variable defines where in the source tree the output will be generated.
By default the image assumes ./target. If its another directory we need to specify the option.

A more complete version of the previous example would then be:

	sti build -e "OUTPUT_DIR=path/to/module/target,MAVEN_ARGS=install -pl some.groupId:some.artifactId -am" <git repo url> dhirajsb/fuse-sti <target image name>

### Real world examples:

Using sti:

	sti build git://github.com/dhirajsb/camel-hello-world dhirajsb/fuse-sti dhirajsb/camel-hello-world --loglevel=5
	sti build git://github.com/dhirajsb/hawtapp-camel-hello-world dhirajsb/fuse-sti dhirajsb/hawtapp-camel-hello-world --loglevel=5

Using oc new-app:

    oc new-app --strategy=source dhirajsb/fuse-sti~git://github.com/dhirajsb/camel-hello-world
    oc new-app --strategy=source dhirajsb/fuse-sti~git://github.com/dhirajsb/hawtapp-camel-hello-world