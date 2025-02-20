# Setting up CoreOS CI

First, create the Jenkins base infra:

```
oc new-app --file=manifests/jenkins.yaml \
  --param=STORAGE_CLASS_NAME=ocs-storagecluster-ceph-rbd
```

Use the `NAMESPACE` parameter if you're not targeting one
named `coreos-ci`.

Create the JCasC configmap:

```
oc create configmap jenkins-casc-cfg --from-file=jenkins/config
```

Create the `github-oauth` secret using `oc create -f`. The
manifest is available in BitWarden.

Create the `github-webhook-shared-secret` secret using `oc
create -f`. The manifest is available in BitWarden.

Create the CoreOS Bot (coreosbot) GitHub token secret (this
corresponds to the "CoreOS CI" token of coreosbot, with just
`public_repo` and `admin:repo_hook`; these creds are
available in BitWarden):

```
apiVersion: v1
kind: Secret
metadata:
  name: github-coreosbot-token
stringData:
  token: TOKEN
```

Create the pipeline configmap:

```
oc new-app --file=manifests/configmap.yaml \
    --param "JENKINS_JOBS_URL=https://github.com/coreos/coreos-ci"
```

If working on your own fork/branch, you can point the
`JENKINS_JOBS_URL` and `JENKINS_JOBS_REF` parameters to
override the repo in which to look for jobs, and/or
`JENKINS_S2I_URL` and `JENKINS_S2I_REF` to override the repo
in which to look for the Jenkins S2I configuration.

Now we can set up the Jenkins S2I builds. We use the same
settings as the FCOS pipeline to ensure that the environment
is the same (notably, Jenkins and plugin versions):

```
oc process -l app=coreos-ci \
    -f https://raw.githubusercontent.com/coreos/fedora-coreos-pipeline/main/manifests/jenkins-s2i.yaml | oc create -f -
```

Then start a build:

```
oc start-build jenkins
```

And that's it! It's already set up so that jobs will be
created on first boot, etc...

## GitHub OAuth

If moving to a new location, you'll need an owner of the
`coreos` GitHub org to update the callback URL of the CoreOS
CI GitHub app to point to the new Jenkins instance as
described in <https://plugins.jenkins.io/github-oauth/>. For
example:

https://jenkins-coreos-ci.apps.ocp.fedoraproject.org/securityRealm/finishLogin
