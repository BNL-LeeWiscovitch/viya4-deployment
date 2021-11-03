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
  --volume /home/lee.wiscovitch/viya4-deployment/deployments:/data \
  --volume /home/lee.wiscovitch/viya4-deployment/deployments/viya-eks/viya-ns/ansible-vars-iac.yaml:/config/config \
  --volume /home/lee.wiscovitch/viya4-iac-aws/terraform.tfstate:/config/tfstate \
  --volume /home/lee.wiscovitch/.ssh/id_rsa:/config/jump_svr_private_key \
  viya4-deployment --tags "baseline,viya,cluster-logging,cluster-monitoring,viya-monitoring,install"
```

### updated run

seems like the above is working, but since I'm running it through vscode remote ssh syntax highlighting isn't working...plus didn't see a vault prompt but no failures yet either

it completed, but not as expected:

```txt
Running: ansible-playbook -e BASE_DIR=/data -e CONFIG=/config/config -e JUMP_SVR_PRIVATE_KEY=/config/jump_svr_private_key -e TFSTATE=/config/tfstate --tags baseline,viya,cluster-logging,cluster-monitoring,viya-monitoring,install playbooks/playbook.yaml

PLAY [localhost] ***************************************************************
Wednesday 03 November 2021  03:08:23 +0000 (0:00:00.018)       0:00:00.018 **** 

TASK [Gathering Facts] *********************************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.957)       0:00:00.976 **** 

TASK [global tmp dir] **********************************************************
changed: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.322)       0:00:01.298 **** 
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.053)       0:00:01.352 **** 

TASK [common : Load config file] ***********************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.047)       0:00:01.399 **** 
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.033)       0:00:01.433 **** 

TASK [common : Parse tfstate] **************************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.049)       0:00:01.482 **** 
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.035)       0:00:01.518 **** 

TASK [common : Add nat ip to LOADBALANCER_SOURCE_RANGES] ***********************
ok: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.048)       0:00:01.566 **** 

TASK [common : tfstate - nfs endpoint] *****************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:24 +0000 (0:00:00.044)       0:00:01.610 **** 

TASK [common : tfstate - nfs path] *********************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.046)       0:00:01.657 **** 

TASK [common : tfstate - export kubeconfig] ************************************
changed: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.645)       0:00:02.302 **** 

TASK [common : tfstate - kubeconfig var] ***************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.043)       0:00:02.346 **** 

TASK [common : tfstate - provider] *********************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.044)       0:00:02.391 **** 
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.036)       0:00:02.428 **** 

TASK [common : tfstate - cluster name] *****************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.044)       0:00:02.472 **** 

TASK [common : tfstate - postgres servers] *************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.045)       0:00:02.518 **** 

TASK [common : tfstate - cluster autoscaler account] ***************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.048)       0:00:02.566 **** 

TASK [common : tfstate - cluster autoscaler location] **************************
ok: [localhost]
Wednesday 03 November 2021  03:08:25 +0000 (0:00:00.044)       0:00:02.610 **** 

TASK [common : tfstate - cluster node pool mode] *******************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.045)       0:00:02.655 **** 

TASK [common : tfstate - jump server] ******************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.042)       0:00:02.698 **** 

TASK [common : tfstate - jump user] ********************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.043)       0:00:02.742 **** 

TASK [common : tfstate - jump rwx filestore path] ******************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.043)       0:00:02.785 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.036)       0:00:02.821 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.048)       0:00:02.869 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:02.904 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:02.940 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:02.976 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.011 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.037)       0:00:03.049 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.057)       0:00:03.107 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.062)       0:00:03.169 **** 

TASK [common : set_fact] *******************************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.205 **** 

TASK [common : Set DEPLOY_DIR] *************************************************
ok: [localhost]
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.039)       0:00:03.244 **** 
included: /viya4-deployment/roles/common/tasks/migrations.yaml for localhost
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.050)       0:00:03.295 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.034)       0:00:03.330 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.365 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.400 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.435 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.033)       0:00:03.469 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.044)       0:00:03.514 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.034)       0:00:03.548 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.032)       0:00:03.580 **** 
Wednesday 03 November 2021  03:08:26 +0000 (0:00:00.035)       0:00:03.616 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.034)       0:00:03.651 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.033)       0:00:03.684 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.036)       0:00:03.720 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.034)       0:00:03.755 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.035)       0:00:03.791 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.037)       0:00:03.828 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.035)       0:00:03.863 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.036)       0:00:03.900 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.034)       0:00:03.934 **** 
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.082)       0:00:04.016 **** 

TASK [jump-server : jump-server - add host] ************************************
changed: [localhost]
Wednesday 03 November 2021  03:08:27 +0000 (0:00:00.056)       0:00:04.072 **** 

TASK [jump-server : jump-server - lookup groups] *******************************
ok: [localhost -> 18.224.113.237]
Wednesday 03 November 2021  03:08:28 +0000 (0:00:01.525)       0:00:05.598 **** 
Wednesday 03 November 2021  03:08:28 +0000 (0:00:00.021)       0:00:05.620 **** 

TASK [jump-server : jumps-server - group nogroup] ******************************
ok: [localhost]
Wednesday 03 November 2021  03:08:29 +0000 (0:00:00.035)       0:00:05.656 **** 

TASK [jump-server : jump-server - create folders] ******************************
changed: [localhost -> 18.224.113.237] => (item=bin)
changed: [localhost -> 18.224.113.237] => (item=homes)
changed: [localhost -> 18.224.113.237] => (item=data)
changed: [localhost -> 18.224.113.237] => (item=astores)
Wednesday 03 November 2021  03:08:30 +0000 (0:00:01.225)       0:00:06.882 **** 
Wednesday 03 November 2021  03:08:30 +0000 (0:00:00.060)       0:00:06.943 **** 
included: /viya4-deployment/roles/baseline/tasks/nfs-subdir-external-provisioner.yaml for localhost
Wednesday 03 November 2021  03:08:30 +0000 (0:00:00.055)       0:00:06.998 **** 

TASK [baseline : Remove deprecated nfs-client-provisioner] *********************
ok: [localhost]
Wednesday 03 November 2021  03:08:31 +0000 (0:00:00.771)       0:00:07.770 **** 

TASK [baseline : Remove deprecated efs-provisioner] ****************************
ok: [localhost]
Wednesday 03 November 2021  03:08:31 +0000 (0:00:00.339)       0:00:08.110 **** 

TASK [baseline : Remove deprecated efs-provisioner namespace] ******************
ok: [localhost]
Wednesday 03 November 2021  03:08:33 +0000 (0:00:01.634)       0:00:09.745 **** 

TASK [baseline : Deploy nfs-subdir-external-provisioner] ***********************
changed: [localhost]
Wednesday 03 November 2021  03:08:43 +0000 (0:00:10.252)       0:00:19.997 **** 
included: /viya4-deployment/roles/baseline/tasks/ingress-nginx.yaml for localhost
Wednesday 03 November 2021  03:08:43 +0000 (0:00:00.044)       0:00:20.042 **** 
Wednesday 03 November 2021  03:08:43 +0000 (0:00:00.021)       0:00:20.063 **** 

TASK [baseline : Deploy ingress-nginx] *****************************************
changed: [localhost]
Wednesday 03 November 2021  03:09:13 +0000 (0:00:30.329)       0:00:50.393 **** 
Wednesday 03 November 2021  03:09:13 +0000 (0:00:00.026)       0:00:50.419 **** 
Wednesday 03 November 2021  03:09:13 +0000 (0:00:00.023)       0:00:50.442 **** 
included: /viya4-deployment/roles/baseline/tasks/cluster-autoscaler.yaml for localhost
Wednesday 03 November 2021  03:09:13 +0000 (0:00:00.042)       0:00:50.485 **** 

TASK [baseline : Deploy cluster-autoscaler] ************************************
changed: [localhost]
Wednesday 03 November 2021  03:09:19 +0000 (0:00:05.882)       0:00:56.368 **** 
included: /viya4-deployment/roles/baseline/tasks/cert-manager.yaml for localhost
Wednesday 03 November 2021  03:09:19 +0000 (0:00:00.043)       0:00:56.411 **** 

TASK [baseline : Deploy cert-manager] ******************************************
changed: [localhost]
Wednesday 03 November 2021  03:09:40 +0000 (0:00:21.130)       0:01:17.541 **** 
included: /viya4-deployment/roles/baseline/tasks/metrics-server.yaml for localhost
Wednesday 03 November 2021  03:09:40 +0000 (0:00:00.034)       0:01:17.576 **** 

TASK [baseline : Check for metrics service] ************************************
ok: [localhost]
Wednesday 03 November 2021  03:09:42 +0000 (0:00:01.705)       0:01:19.281 **** 

TASK [baseline : Deploy metrics-server] ****************************************
changed: [localhost]
Wednesday 03 November 2021  03:10:01 +0000 (0:00:18.898)       0:01:38.181 **** 
Wednesday 03 November 2021  03:10:01 +0000 (0:00:00.099)       0:01:38.281 **** 

TASK [monitoring : v4m - download] *********************************************
changed: [localhost]
Wednesday 03 November 2021  03:10:04 +0000 (0:00:02.397)       0:01:40.678 **** 

TASK [monitoring : v4m - add storageclass] *************************************
changed: [localhost]
Wednesday 03 November 2021  03:10:05 +0000 (0:00:01.326)       0:01:42.004 **** 
included: /viya4-deployment/roles/monitoring/tasks/cluster-monitoring.yaml for localhost
Wednesday 03 November 2021  03:10:05 +0000 (0:00:00.036)       0:01:42.041 **** 

TASK [monitoring : cluster-monitoring - create userdir] ************************
changed: [localhost]
Wednesday 03 November 2021  03:10:05 +0000 (0:00:00.219)       0:01:42.260 **** 

TASK [monitoring : cluster-monitoring - lookup existing credentials] ***********
ok: [localhost]
Wednesday 03 November 2021  03:10:06 +0000 (0:00:01.030)       0:01:43.291 **** 
Wednesday 03 November 2021  03:10:06 +0000 (0:00:00.022)       0:01:43.313 **** 

TASK [monitoring : cluster-monitoring - output credentials] ********************
ok: [localhost] => {
    "msg": [
        "Grafana - username: admin, password: MZpsZXrabNDmaYQgVzha"
    ]
}
Wednesday 03 November 2021  03:10:06 +0000 (0:00:00.032)       0:01:43.345 **** 

TASK [monitoring : cluster-monitoring - user values] ***************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ansible.errors.AnsibleUndefinedVariable: prometheus.{{ V4M_BASE_DOMAIN }}: {{ V4_CFG_BASE_DOMAIN }}: 'V4_CFG_BASE_DOMAIN' is undefined
fatal: [localhost]: FAILED! => {"changed": false, "msg": "AnsibleUndefinedVariable: prometheus.{{ V4M_BASE_DOMAIN }}: {{ V4_CFG_BASE_DOMAIN }}: 'V4_CFG_BASE_DOMAIN' is undefined"}

PLAY RECAP *********************************************************************
localhost                  : ok=45   changed=12   unreachable=0    failed=1    skipped=35   rescued=0    ignored=0   

Wednesday 03 November 2021  03:10:06 +0000 (0:00:00.041)       0:01:43.386 **** 
=============================================================================== 
baseline : Deploy ingress-nginx ---------------------------------------- 30.33s
baseline : Deploy cert-manager ----------------------------------------- 21.13s
baseline : Deploy metrics-server --------------------------------------- 18.90s
baseline : Deploy nfs-subdir-external-provisioner ---------------------- 10.25s
baseline : Deploy cluster-autoscaler ------------------------------------ 5.88s
monitoring : v4m - download --------------------------------------------- 2.40s
baseline : Check for metrics service ------------------------------------ 1.71s
baseline : Remove deprecated efs-provisioner namespace ------------------ 1.63s
jump-server : jump-server - lookup groups ------------------------------- 1.53s
monitoring : v4m - add storageclass ------------------------------------- 1.33s
jump-server : jump-server - create folders ------------------------------ 1.23s
monitoring : cluster-monitoring - lookup existing credentials ----------- 1.03s
Gathering Facts --------------------------------------------------------- 0.96s
baseline : Remove deprecated nfs-client-provisioner --------------------- 0.77s
common : tfstate - export kubeconfig ------------------------------------ 0.65s
baseline : Remove deprecated efs-provisioner ---------------------------- 0.34s
global tmp dir ---------------------------------------------------------- 0.32s
monitoring : cluster-monitoring - create userdir ------------------------ 0.22s
monitoring role - cluster ----------------------------------------------- 0.10s
jump-server role -------------------------------------------------------- 0.08s
```

skipped a lot of things...killed cluster (terraform destroy) and will retry tomorrow.
