# nameko-example

## todos
 - [x] nameko hello world
 - [x] put it into docker image
 - [x] deploy them to kubernetes via helm
 - [ ] ~~upgrade them with fluxcd~~ i could not make automated policy works with unpacked helm chats.
   - [x] double check.
   - [x] check what happen if I add fluxcd.io via kubectl edit instead of having tha in the manifest in git repo
 - [ ] use HelmRelease Custom Resource instead unpacked charts.
 


## Observations

### fluxcd.io annotations
Workload policy loose automation, when I commented out fluxcd.io manifestation. It was expected.
```
#  annotations:
#    fluxcd.io/automated: "true"
#    fluxcd.io/tag.hello-app: semver:*
```
![workloads](https://raw.githubusercontent.com/olivernadj/nameko-example/master/lost-automation.png "Workload with and without annotation")

Although editing `kubectl edit deployment hello-depl` and add back fluxcd.io annotations has no effect. 
That was unexpected. It means flux is threats git repo as a source of truth and does not scan actual deployment instances. 

### POLICY automated
As I expected flux indeed replace the image with a newest tag and also push back the new tag into git repo.


### Unpacked helm charts
Unpacked helm charts are pretty much ignored, by fluxcd.

 
### fluxctl install 
 
```bash
export GHUSER="olivernadj"
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/nameko-example.git \
--git-path=release \
--namespace=flux | kubectl apply -f - 

kubectl -n flux rollout status deployment/flux
```

troubleshooting 
https://github.com/fluxcd/flux/issues/2517
```
- --registry-exclude-image=*
- --registry-include-image=docker.io/olivernadj/*
```

### add deploy keys

```
export FLUX_FORWARD_NAMESPACE="flux" 
fluxctl identity
```
https://github.com/olivernadj/nameko-example/settings/keys


### check current images
```bash
kubectl get pods --all-namespaces -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```