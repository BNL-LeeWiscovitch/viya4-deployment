install and confirm docker is working

<https://docs.docker.com/engine/install/centos/>

make sure to perform the post-install tasks so normal user can access it

<https://docs.docker.com/engine/install/linux-postinstall/>

clone the repo

```shell
git clone git@github.com:BNL-LeeWiscovitch/viya4-deployment.git && \
cd viya4-deployment
```

create docker container

```shell
docker build -t viya4-deployment .
```

lots of apt/deb installs, just look for errors

create folder structure in repo:

```txt
<base_dir>            <- parent directory
  /<cluster>          <- folder per cluster
    /<namespace>      <- folder per namespace
      /site-config    <- location for all customizations
        ...           <- folders containing user defined customizations
```

make new directory in repo named "deployments", this is known as the <base_dir>

create "viya-eks" for the <cluster> folder (unclear if it needs to match the existing eks cluster or not...we'll find out!).

create "viya-ns" for the <namespace> folder

create "site-config" folder in trhe <namespace> folder

you should have the following layout now:

```txt
/deployments          <- parent directory
  /viya-eks           <- folder per cluster
    /viya-ns          <- folder per namespace
      /site-config    <- location for all customizations
        ...           <- folders containing user defined customizations
```

copy /examples/ansible-vars-iac.yaml to /deployments/viya-eks/viya-ns/ansible-vars-iac.yaml

>that file will consume the tfstate file, getting as much of the values as it can automatically

# ansible vault

since we have sensitive information, and can't make the repo private, need to make an ansible vault yaml that contains the sensitive information that we can reference in the various files

since you can't run ansible on windows, need to make/edit the vault on admin-pl-cli02


sas order api

<https://apiportal.sas.com/my-apps/new-app>

created new app called "viya-eks" and enabled the sas viya orders api and saved it.

this created a new api key, stored in bitwarden as "viya-eks-sas-api"
