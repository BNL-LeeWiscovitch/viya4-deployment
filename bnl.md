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

create new vault at /deployments/viya-eks/viya-ns

```shell
ansible-vault create /deployments/viya-eks/viya-ns/vault.yml
```

>password is stored in bitwarden as "viya-eks-ansible-vault"

any future edits has to be done on admin-pl-cli002 (or another instance with github/ansible)

sas order api

<https://apiportal.sas.com/my-apps/new-app>

created new app called "viya-eks" and enabled the sas viya orders api and saved it.

this created a new api key, stored in bitwarden as "viya-eks-sas-api", stored in ansible vault as well.

<<<<<<< HEAD
## test run

this is just to install the basics for viya (no monitoring/logging, as those were failing)

```txt
docker run --rm \
  --group-add root \
  --user $(id -u):$(id -g) \
  --volume /home/lee.wiscovitch/viya4-deployment/deployments:/data \
  --volume /home/lee.wiscovitch/viya4-deployment/deployments/viya-eks/viya-ns/ansible-vars-iac.yaml:/config/config \
  --volume /home/lee.wiscovitch/viya4-iac-aws/terraform.tfstate:/config/tfstate \
  --volume /home/lee.wiscovitch/.ssh/id_rsa:/config/jump_svr_private_key \
  viya4-deployment --tags "baseline,viya,install" \
  --vault-password-file /home/lee.wiscovitch/.vault.txt \
  -vvvv
```

### fail

```txt
Running: ansible-playbook -e BASE_DIR=/data -e CONFIG=/config/config -e JUMP_SVR_PRIVATE_KEY=/config/jump_svr_private_key -e TFSTATE=/config/tfstate --tags baseline,viya,install --vault-password-file /home/lee.wiscovitch/.vault.txt -vvvv playbooks/playbook.yaml
ansible-playbook 2.10.15
  config file = /viya4-deployment/ansible.cfg
  configured module search path = ['/usr/share/ansible', '/viya4-deployment/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.8.10 (default, Sep 28 2021, 16:10:42) [GCC 9.3.0]
Using /viya4-deployment/ansible.cfg as config file
ERROR! The vault password file /home/lee.wiscovitch/.vault.txt was not found
```


