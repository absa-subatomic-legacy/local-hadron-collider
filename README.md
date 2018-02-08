# Local Hadron Collider

Resources for getting a local [Subatomic](https://github.com/absa-subatomic)
development environment up and running.

## Developer setup

Follow the below steps to get a local Subatomic environment ready for local development:

### 1. Install minishift

Follow the [instructions to install](https://docs.openshift.org/latest/minishift/getting-started/installing.html)
minishift for your platform.

### 2. Clone this project

Clone this project locally:

```console
$ git clone https://github.com/absa-subatomic/local-hadron-collider.git
$ cd local-hadron-collider
```

### 3. Enable minishift addons

A Subatomic minishift [addon](https://docs.openshift.org/latest/minishift/using/addons.html) is used to configure the routing for Subatomic.

There is a [preconfigured certificate chain](minishift-addons/subatomic/certs) for the `*.subatomic.local` domain. The [Subatomic minishift addon](minishift-addons/subatomic/subatomic.addon) configures the default routing subdomain to `subatomic.local` and replaces the existing default router certificate with the `*.subatomic.local` wildcard certificate and certificate chain.

Install and enable the Subatomic addon and enable `admin-user` addon with the following:

```console
$ minishift addons enable admin-user --profile subatomic-local
Add-on 'admin-user' enabled
$ minishift addons install minishift-addons/subatomic --profile subatomic-local
Addon 'subatomic' installed
$ minishift addons enable subatomic --profile subatomic-local
Add-on 'subatomic' enabled
```

### 4. Start a Subatomic minishift instance

Run a new minishift instance for local development:

```console
$ minishift start \
  --profile subatomic-local \
  --cpus 4 --memory 6144MB --disk-size 60GB
-- Starting profile 'subatomic-local'
...
-- Applying addon 'subatomic':
Replacing default router certificate for *.subatomic.local....
Setting the routing subdomain to 'subatomic.local'..

The Subatomic routing configurations have been successfully completed 👍
Due to updating the `master-config.yaml` you must restart your minishift instance

Restart minishift by executing the following command *now*:

    minishift openshift restart
-- Applying addon 'admin-user':..
```

#### 4.1 Restart minishift

For the routing subdomain change you must restart your minishift instance:

```console
$ minishift openshift restart
Restarting OpenShift
```

### 5. Configure local DNS

To resolve the `subatomic.local` domain you must setup your local machine to resolve to the minishift IP.

First get the minishift IP with:

```console
$ minishift ip
192.168.64.15
```

#### dnsmasq

> This is the recommended solution to local DNS

To configure dnsmasq to resolve to your minishift IP, add the following to your `dnsmasq.conf`:

```
# IP of the HAProxy router: minishift ip
address=/subatomic.local/192.168.64.15
```

Restart dnsmasq and you should be able to `curl` a non-existent route:

```console
$ curl -I nothing.subatomic.local
HTTP/1.0 503 Service Unavailable
Pragma: no-cache
Cache-Control: private, max-age=0, no-cache, no-store
Connection: close
Content-Type: text/html
```

You should get a `503 Service Unavailable` response.

#### `/etc/hosts`

You could also manually add entries into your `/etc/hosts` for the various routes you expose.

### 6. Install the Subatomic infratructure components

Subatomic needs the following components installed:

* Bitbucket Server
* Nexus

In addition, the Jenkins and S2I images beed to be built and available.

All this is accomplished by running the [OpenShift Applier](https://github.com/redhat-cop/casl-ansible/tree/master/roles/openshift-applier). The applier will add the various resources required to build and deploy the above.

> The OpenShift Applier requires that Ansible be installed. Please follow the [installation guide](http://docs.ansible.com/ansible/latest/intro_installation.html)
to install for your platform.

#### 6.1 Log in with the `oc` CLI tool

The OpenShift Applier will use the current logged in context for your minishift instance. So first, login using the `oc` CLI tool.

Login into the OpenShift console UI as the `admin` user (`admin` / `admin`) and get your token by clicking on _Copy Login Command_:

![Get token from OpenShift console](/docs/images/openshift-console-token.png)

Now login using the `oc` CLI tool:

```console
$ eval $(minishift oc-env)
$ oc login https://192.168.64.15:8443 --token=UaUbbHWlHTul8mecJuM31mJQfmid7x5dV0fIecihW8c
...
```

#### 6.2 Clone the casl-ansible repository

The OpenShift Applier is a playbook implemented in the `casl-ansible` project. Therefore, you need to clone this project at the same directory level as the `local-hadron-collider` directory.

```console
$ cd ..
$ https://github.com/redhat-cop/casl-ansible
...
```

#### 6.3 Run the OpenShift Applier

Now change back into your `local-hadron-collider` directory and run with:

```console
$ cd local-hadron-collider
$ ansible-playbook -i inventory/hosts ../casl-ansible/playbooks/openshift-cluster-seed.yml --connection=local
...
```

Two new projects will be created:

* Subatomic - Project for Tempalates, Image Streams etc.

```console
$ oc get builds --namespace subatomic
NAME                               TYPE      FROM          STATUS                       STARTED              DURATION
jdk8-maven3-newrelic-subatomic-3   Docker    Git@f812447   Complete                     4 minutes ago        15m47s
jenkins-slave-maven-subatomic-1    Docker    Git@2129cfe   Complete                     31 minutes ago       15m28s
jenkins-subatomic-1                Docker    Git@2129cfe   Complete                     31 minutes ago       22m4s

$ oc get imagestreams --namespace subatomic
NAME                             DOCKER REPO                                                TAGS      UPDATED
jdk8-maven3-newrelic-subatomic   172.30.1.1:5000/subatomic/jdk8-maven3-newrelic-subatomic   2.0       18 minutes ago
jenkins-slave-maven-subatomic    172.30.1.1:5000/subatomic/jenkins-slave-maven-subatomic    2.0       15 minutes ago
jenkins-subatomic                172.30.1.1:5000/subatomic/jenkins-subatomic                2.0       8 minutes ago
```

* Subatomic Infrastructure - Project where Bitbucket Server and Nexus are deployed too

```console
$ oc get deploymentconfigs --namespace subatomic-infra
NAME               REVISION   DESIRED   CURRENT   TRIGGERED BY
bitbucket-server   1          1         1         config,image(bitbucket-server:5.4.6)
nexus              1          1         1         config,image(nexus:2.14.6-02)
```

Now wait for the various deployments and builds to complete.

> ⚠️ This can take a long time depending on your internet connection.

### 7. Setup Bitbucket Server

Once the Bitbucket Server deployment has completed successfully, you need to complete the setup via the web interface.

Open your Bitbucket Server instance by clicking this link: https://bitbucket.subatomic.local/

Choose the `Internal` database and when prompted for a license, either get a trial license or use a Developer License from your Atlassian technical contact.

![Bitbucket Server setup](/docs/images/bitbucket-setup-1.gif)

Next, configure the administrator user as follows:

![Bitbucket Server setup](/docs/images/bitbucket-setup-2.gif)

| Field         | Value |
| ------------- | ------ |
|Username       | subatomic |
|Full name      | Subatomic |
|Email          | subatomic@local |
|Password       | subatomic |

