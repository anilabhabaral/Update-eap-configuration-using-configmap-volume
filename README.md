# By modifying the `standalone-openshift.xml`

1. Get the existing configuration:
```
oc exec $POD_NAME cat /opt/eap/standalone/configuration/standalone-openshift.xml > standalone-openshift.xml
```
2. Edit the file as needed.
3. Set ENV CONFIG_IS_FINAL=true
```
$ oc set env dc $DC_NAME CONFIG_IS_FINAL=true
```
4. Mount the updated file as a ConfigMap:
```
oc create cm standalone --from-file=standalone-openshift.xml
```
5. Add the ConfigMap to your DeploymentConfig
```
oc set volume dc $DC_NAME --add --name=standalone --mount-path=/opt/eap/standalone/configuration/standalone-openshift.xml --sub-path=standalone-openshift.xml --type=configmap --configmap-name=standalone
```

# By using CLI script

1. Create a CLI script(.cli) file with the required CLI commands. For example:
```
$ cat extensions/extensions.cli
embed-server --std-out=echo --server-config=standalone-openshift.xml
batch
/subsystem=io/worker=default:write-attribute(name=task-max-threads,value=10)
run-batch
quit
```

2. Create a script file(.sh) to excute the above CLI command. For example:
```
$ cat extensions/postconfigure.sh
#!/bin/sh
echo "Executing postconfigure.sh"
$JBOSS_HOME/bin/jboss-cli.sh --file=$JBOSS_HOME/extensions/extensions.cli
```
3. Mount the .cli and .sh file as a ConfigMap:
```
$ oc create configmap jboss-cli --from-file=postconfigure.sh=extensions/postconfigure.sh --from-file=extensions.cli=extensions/extensions.cli
```
5. Add the ConfigMap to your DeploymentConfig
```
$ oc set volume deployment $DEPLOYMENT_NAME --add --name=jboss-cli -m /opt/eap/extensions -t configmap --configmap-name=jboss-cli --default-mode='0755'
```
