# ocs-disk-gather
Gather disk information from OCP4 nodes for OCS 4 local storage setup 

- there are 3 versions of disk gatherer  currently available: 
  1. `ocs-disk-gatherer.yaml`
      Everyting is created in default namespace which needs no scc as default project does not apply those yet (this might change in future) and pulls the image from legacy and deprecated registry.access.redhat.com and hence requires no pull secret. This might change in future as well 
  2. `ocs-disk-gatherer-sha-no-secret.yaml`
      same as 1. but uses sha instead of tag (no tag implies latest) which might be needed in some circomstances
  3. `ocs-disk-gatherer.yaml` 
      This yaml creates 
      - a namespace called `ocs-disk-gatherer`
      - a service account called `ocs-disk-gatherer-sa`
      - a scc called `ocs-disk-gatherer-sa` to be attached to the service account
      - a pull secret to fetch the RHEL 8 ubi image from the registry 
        you need to get a pull secret from https://access.redhat.com/terms-based-registry/ and add it to the ubi-pull-secret section in the `ocs-disk-gatherer.yaml`


- all of the three choices from above create a daemon set called `ocs-disk-gatherer` that automatically deploys on every node labeld with `cluster.ocs.openshift.io/openshift-storage:""`
  The ds will start a loop refreshing all 10 minutes the disks it found, output to look like this:

* check deployment is running 
~~~
# oc get ds -o wide
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                 AGE   CONTAINERS   IMAGES                                SELECTOR
ocs-disk-gatherer   3         3         3       3            3           cluster.ocs.openshift.io/openshift-storage=   37s   collector    registry.access.redhat.com/ubi8/ubi   name=ocs-disk-gatherer
# 
~~~

* check pods running
~~~
# oc get po -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP            NODE                              NOMINATED NODE   READINESS GATES
ocs-disk-gatherer-4bjg5   1/1     Running   0          42s   10.131.2.12   cluster-jfb49-workerocs-0-plsbc   <none>           <none>
ocs-disk-gatherer-bhhlb   1/1     Running   0          42s   10.129.4.15   cluster-jfb49-workerocs-0-lnq2p   <none>           <none>
ocs-disk-gatherer-nwhhn   1/1     Running   0          42s   10.128.4.21   cluster-jfb49-workerocs-0-szp97   <none>           <none>
# 
~~~

* check logs on a certain pod
~~~
# oc logs ocs-disk-gatherer-bhhlb
          # NODE:cluster-jfb49-workerocs-0-lnq2p
          # sda : 120G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_143fdada-9a4e-47de-9
          # sdb : 25G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_c6c84893-9de0-402c-a
          # sdc : 1.2T
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_2bfc0ad1-812d-4702-8
 -------------------------------------------
# 
~~~

* check logs on all pods running at once
~~~
# oc logs -l name=ocs-disk-gatherer --tail=-1 --since=10m (or # oc logs -l name=ocs-disk-gatherer --tail=-1 --since=10m -n ocs-disk-gatherer)
          # NODE:cluster-jfb49-workerocs-0-lnq2p
          # sda : 120G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_143fdada-9a4e-47de-9
          # sdb : 25G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_c6c84893-9de0-402c-a
          # sdc : 1.2T
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_2bfc0ad1-812d-4702-8
 -------------------------------------------
          # NODE:cluster-jfb49-workerocs-0-szp97
          # sda : 120G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_26e6a4c3-5faf-4052-9
          # sdb : 1.2T
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_da04fbd6-dc2f-4a31-b
          # sdc : 25G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_1b5eeeae-f9a2-467f-9
 -------------------------------------------
          # NODE:cluster-jfb49-workerocs-0-plsbc
          # sda : 120G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_e5f05350-bddb-409e-a
          # sdb : 1.2T
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_afa53a80-a2a0-47d6-9
          # sdc : 25G
        - /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_e690e3a2-4bc0-4deb-b
 -------------------------------------------
#
~~~

* run one of the following to get the version of your choics:
  - `wget 
  - `wget
  - `wget 
  In case you've chosen the last one, don't forget to to modify the yaml and put your secret in :
~~~
# vim ocs-disk-gatherer-own-project.yaml
...
---
apiVersion: v1
kind: Secret
metadata:
  name: ubi-pull-secret
  namespace: ocs-disk-gatherer
data:
  .dockerconfigjson: <redacted, fetch from https://access.redhat.com/terms-based-registry/ >
type: kubernetes.io/dockerconfigjson
---
...
~~~
  then run  `oc create -f < file >`
 
 
