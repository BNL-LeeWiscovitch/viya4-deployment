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

need to copy the 

probably going to fail, but let's see where/how:

```txt
docker run --rm \
  --group-add root \
  --user $(id -u):$(id -g) \
  --volume $HOME/deployments:/data \
  --volume $HOME/deployments/viya-eks/viya-ns/ansible-vars-iac.yaml:/config/config \
  --volume /home/lee.wiscovitch/viya4-iac-aws/terraform.tfstate:/config/tfstate \
  --volume /home/lee.wiscovitch/.ssh/id_rsa:/config/jump_svr_private_key \
  viya4-deployment --tags "baseline,viya,cluster-logging,cluster-monitoring,viya-monitoring,install"
```

### fail

```txt

Running: ansible-playbook -e BASE_DIR=/data -e CONFIG=/config/config -e JUMP_SVR_PRIVATE_KEY=/config/jump_svr_private_key -e TFSTATE=/config/tfstate --tags baseline,viya,cluster-logging,cluster-monitoring,viya-monitoring,install playbooks/playbook.yaml

PLAY [localhost] ***************************************************************
Tuesday 02 November 2021  20:47:17 +0000 (0:00:00.021)       0:00:00.021 ****** 

TASK [Gathering Facts] *********************************************************
ok: [localhost]
Tuesday 02 November 2021  20:47:18 +0000 (0:00:01.016)       0:00:01.037 ****** 

TASK [global tmp dir] **********************************************************
changed: [localhost]
Tuesday 02 November 2021  20:47:19 +0000 (0:00:00.354)       0:00:01.392 ****** 
Tuesday 02 November 2021  20:47:19 +0000 (0:00:00.050)       0:00:01.443 ****** 

TASK [common : Load config file] ***********************************************
fatal: [localhost]: FAILED! => {"ansible_facts": {}, "ansible_included_var_files": [], "changed": false, "message": "an error occurred while trying to read the file '/config/config': [Errno 21] Is a directory: b'/config/config'"}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

Tuesday 02 November 2021  20:47:19 +0000 (0:00:00.045)       0:00:01.488 ****** 
=============================================================================== 
Gathering Facts --------------------------------------------------------- 1.02s
global tmp dir ---------------------------------------------------------- 0.35s
common role ------------------------------------------------------------- 0.05s
common : Load config file ----------------------------------------------- 0.05s
```

is that because of the vault? it didn't prompt for password
