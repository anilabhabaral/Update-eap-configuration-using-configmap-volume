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
