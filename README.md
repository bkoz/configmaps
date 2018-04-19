# Simple OpenShift configmap
## This example creates a configmap from a directory of files then gets mounted to the pod as a volume.

It is based on an [example](https://docs.openshift.com/container-platform/3.9/dev_guide/configmaps.html#configmaps-creating-from-directories) from the OpenShift Developer's Guide.

Create a new project.

~~~shell
oc new-project cmaps
~~~

Create the directory and a few configuration files.

~~~shell
mkdir example-files
~~~

Copy and paste the following blocks into bash:

~~~shell
cat >> example-files/game.properties<<EOF
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
EOF
~~~

~~~shell
cat >> example-files/ui.properties<<EOF
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
EOF
~~~

Create the configmap from the directory.

~~~shell
oc create configmap game-config --from-file=example-files/
~~~

Confirm the configmap was created.

~~~shell
oc get configmap
~~~

Expected Output:

~~~shell
NAME          DATA      AGE
game-config   2         10s
~~~

Create a sample applciation.

~~~shell
oc new-app cakephp-mysql-example
~~~

Inject the configmap into the deployment config.

~~~shell
oc get deploymentconfig
oc volume deploymentconfig/cakephp-mysql-example --add --configmap-name=game-config --mount-path=/config
~~~

Expected Output:

~~~shell
deploymentconfig "cakephp-mysql-example" updated
~~~

Wait for the new pod to deploy and verify the configmap is mounted to the pod.

~~~shell
POD_NAME=`oc get pods --selector=name=cakephp-mysql-example --output=custom-columns=NAME:.metadata.name --no-headers`
oc rsh $POD_NAME df
~~~

Expected output:

~~~shell
Filesystem                                                                                          1K-blocks     Used Available Use% Mounted on
...
...
/dev/mapper/vgsys-var                                                                                47162880 12100516  35062364  26% /config
~~~

Confirm one of the configuration files is readable.

~~~shell
oc rsh $POD_NAME cat /config/game.properties
~~~
