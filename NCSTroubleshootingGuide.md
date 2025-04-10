NCS Troubleshooting Guide

Upgrade troubleshooting

After the OS Upgrade passed for all of the nodes, the ISO is removed
from host, and from request. This will cause the request to be treated
as updated one because it is different from previous one, and triggers
update of config files with the new request, which fails.

11.2 Upgrade fails during pre-verification due to the upgrade service is
unable to connect to the ironic server

11.2.1 Problem Description: Error contacting Ironic server: Unable to
establish connection to http://172.31.7.254:6385/v1/nodes:
HTTPConnectionPool(host='172.31.7.254', port=6385): Max retries exceeded
with url: /v1/nodes (Caused by
NewConnectionError('\<urllib3.connection.HTTPConnection object at
0x7fe810dc9290\>: Failed to establish a new connection: \[Errno 113\] No
route to host',)). Attempt 6 of 6 The problem can have several root
causes. One cause is the IRONIC\_URL environment variable used for
connecting to the Ironic server is incorrect.

11.2.2 Solution Workaround: 1. Check what should be the correct value
from the inventory: \[root\@server-name\]\# grep
internal\_lb\_vip\_address
/opt/install/data/cbisclusters/<your cluster name>/postconfig-inv.json
"internal\_lb\_vip\_address": "172.31.7.1" 2. Check if the IP address is
present on the host: \[root\@server-name\]\# ip a \| grep 172.31.7.1
inet 172.31.7.1/24 brd 172.31.7.255 scope global br-provision

Nokia Container Services Release 24.7

DN1000065220 1-0 ©2025 Nokia. Nokia Confidential Information Use subject
to agreed restrictions on disclosure and use.

219

NCS Troubleshooting Guide

Upgrade troubleshooting

3.  Copy the following ansible playbook to a file: --- hosts:
    installation\_controller gather\_facts: false become: true tasks:

-   name: set variable set\_fact: cbis\_clusters\_path:
    /opt/install/data/cbis-clusters/
-   name: remove line lineinfile: line: "source {{ cbis\_clusters\_path
    }}.bm\_env" state: absent path: "{{ item }}" loop:
-   '/root/.bashrc'
-   '/home/cbis-admin/.bashrc'
-   name: delete bm\_env file file: path: "{{ cbis\_clusters\_path
    }}.bm\_env" state: absent
-   name: recreate bm\_env file include\_role: name:
    /usr/share/cbis/cbis-ansible/pre-deploy/ set\_environment\_variables
    vars: env\_path: "{{ cbis\_clusters\_path }}"

4.  Run the playbook: openstack-ansible -b -u cbis-admin
    /root/fix\_bm\_env.yml
5.  Verify that the openstack baremetal command works:
    \[root\@server-name\]\# openstack baremetal node list
    +--------------------------------------+---------------------------+---------------+-------------+--------------------+-------------+
    \| UUID \| Name Instance UUID \| Power State \| Provisioning State
    \| Maintenance \|

Nokia Container Services Release 24.7

DN1000065220 1-0 ©2025 Nokia. Nokia Confidential Information Use subject
to agreed restrictions on disclosure and use.

| 

220

NCS Troubleshooting Guide

Upgrade troubleshooting

+--------------------------------------+---------------------------+---------------+-------------+--------------------+-------------+
\| 0c0a8130-e865-4f9e-8859-cc3998f9e1c5 \| tokyo-monitorbm-0 \| power on
\| active \| True \|

| None

| 38042adf-5a62-4d8b-a6d0-20cac1ec1fb2 \| tokyo-monitorbm-1
| power on
| active
| True
| 

| None

6.  Resume the upgrade.

11.3 Upgrade fails in BCMT upgrade stage on: TASK \[Verify if infra
components are Running\] 11.3.1 Problem When checking the system, a few
pods will be evicted and others in pending state. Check if any of the
nodes have disk pressure using the following command: \#kubectl get node
-o customcolumns=NAME:.metadata.name,TAINT:.spec.taints\[\*\].key \|
grep 'node.kubernetes.io/disk-pressure' On the affected node, check the
usage of /data0: \#df -h /data0

11.3.2 Solution 1. Run the following command to clean docker registry:
\#podman prune image --all -f 2. Resume upgrade.

11.4 Upgrade fails at Worker nodes : Retry file does not exist /root/
CSF-BCMT/ansible/cluster.retry

Nokia Container Services Release 24.7

DN1000065220 1-0 ©2025 Nokia. Nokia Confidential Information Use subject
to agreed restrictions on disclosure and use.

221


