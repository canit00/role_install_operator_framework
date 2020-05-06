##### How to manually deploy an operator
git clone https://github.com/coreos/etcd-operator.git
cd $HOME/etcd-operator
oc new-project etcd-operators
./example/rbac/create_role.sh --namespace=etcd-operators

###### Pod states
example-etcd-cluster-8s9w86f785   0/1       Pending   0         0s
example-etcd-cluster-8s9w86f785   0/1       Pending   0         0s
example-etcd-cluster-8s9w86f785   0/1       Init:0/1   0         0s
example-etcd-cluster-8s9w86f785   0/1       Init:0/1   0         3s
example-etcd-cluster-8s9w86f785   0/1       PodInitializing   0         4s
example-etcd-cluster-8s9w86f785   0/1       Running   0         5s
example-etcd-cluster-8s9w86f785   1/1       Running   0         6s

##### Operator SDK
- Operators written in programming language using Kubernetes API library
- Default: Go
  - Other programming languages possible
    - Example: Java™

- Need to write entire controller logic
  - Requires knowledge of informers, shared informers, work queues for object cache and event handling

**Features of Red Hat Operator SDK**

Features | Use
--------------
High-level APIs and abstractions | Write operational logic more intuitively
Tools for scaffolding and code generation | Bootstrap new project quickly
Extensions | Cover common Operator use cases

##### Ansible Operator
**Overview**
- Provides base Operator image
- Executes Ansible Playbook or role
- Keeps running playbook/role in loop
  - Ensures objects are there
  - Re-creates changed/deleted objects
  - Playbook or role must be idempotent
- Allows creation, but not deletion, of objects in OpenShift
- Adds ownerReferences to all created objects

**Reference**
- [User's Guide](https://sdk.operatorframework.io/)

**Workflow**
- Ansible Operator SDK creates basic structure needed
- Update Ansible content
  - To create objects in role/playbook, use k8s Ansible module
- Build Operator container image using provided Dockerfile
- Push to image registry—e.g., Quay.io
- Update created YAML files

**Installing an Ansible-Based Operator**
- Deploy Ansible-based Operator like any other Operator
  - Create CRD
    - Requires cluster-admin
- Create project
- Create service account
- Create Role or ClusterRole
- Create RoleBinding or ClusterRoleBinding
- Create Deployment

**Testing Outside the Cluster**
- When all prerequisites have been created, SDK can run Operator outside cluster
  - Replaces deployment of deploy/operator.yaml
  - Still watches for CR-created events
  - Eliminates need to rebuild/push/redeploy container image every time
    - Log in to OpenShift
    - From Operator directory:

```
operator-sdk up local
```

##### Helm Operator
**Overview**
- Convert Helm charts into Operator
- No need for Tiller on cluster
- Helm charts needed to be OpenShift-compatible
  - not run as root
  - no other prerequisites
**Reference**
- [Make a k8s op in 15 min with Helm](https://www.openshift.com/blog/make-a-kubernetes-operator-in-15-minutes-with-helm)

Setting up a Helm Operator
- Create a project of type helm
```
operator-sdk new nginx-operator --api-version=example.com/v1alpha1 --kind=Nginx --type=helm
```
**Resources**
- [Helm Operator User's Guide](https://github.com/operator-framework/operator-sdk/blob/v0.4.x/doc/helm/user-guide.md)

##### Install Operator SDK
Download the Operator SDK binary into `/usr/local/bin`:
```
wget https://github.com/operator-framework/operator-sdk/releases/download/v0.6.0/operator-sdk-v0.6.0-x86_64-linux-gnu -O /usr/local/bin/operator-sdk
chmod +x /usr/local/bin/operator-sdk
```
Add the directory `/usr/local/bin` to your **PATH** and save the **PATH** for future logins:
```
export PATH=$PATH:/usr/local/bin
echo "export PATH=$PATH:/usr/local/bin" >> $HOME/.bashrc
```

##### Create Operator from Ansible Roles Using Operator SDK
```
cd $HOME
git clone https://github.com/redhat-gpte-devopsautomation/ansible-operator-roles
cd ansible-operator-roles
git checkout v0.6.0
cd $HOME
```

##### Create Ansible Operator Skeleton
See the [Ansible Operator User's Guide](https://github.com/operator-framework/operator-sdk/blob/v0.6.0/doc/ansible/user-guide.md)

From your home directory, use the operator-sdk binary to create a new Ansible Operator skeleton:
- Use the following values to ensure that your Operator works in the shared cluster:
  - gogs-operator as the Operator name
  - ocp4.freebsd.us/v1alpha1 as the API version
  - Gogs as the kind
  - Use `--skip-git-init` as an additional flag to skip creation of a Git repository

Set ansible as the Operator type and include the option to use a playbook and not just roles.
```
cd $HOME
operator-sdk new gogs-operator --api-version=ocpv4.freebsd.us/v1alpha1 --kind=Gogs --type=ansible --generate-playbook --skip-git-init
```

##### Update Skeleton
The operator-sdk created a skeleton playbook and a skeleton roles directory. But you already have a playbook and roles from the repository you cloned earlier. In this section, you update your skeleton.
```
cd $HOME/gogs-operator
rm -rf roles playbook.yml
mkdir roles
cp -R $HOME/ansible-operator-roles/roles/postgresql-ocp ./roles
cp -R $HOME/ansible-operator-roles/roles/gogs-ocp ./roles
cp $HOME/ansible-operator-roles/playbooks/gogs.yaml ./playbook.yml
```

Examine the `watches.yaml` file:
```
cat ./watches.yaml
```

This is the main configuration for the Ansible Operator. The group, version, and kind come from the command you used to create the skeleton. The playbook uses the playbook.yml filename. In the next step, you see how the file ends up in the /opt/ansible location.

Examine the Dockerfile in the build directory
```
cat build/Dockerfile
```

Note the source image for the container image that is being built and observe that all the Dockerfile does is copy the roles, playbook, and watches files into the container image.

The ansible-operator container image already contains the logic that an Operator needs and simply uses the playbook to set up the application.

The playbook needs to be idempotent (meaning that it must not fail when run a second time) because the Operator keeps running the playbook periodically to ensure that all necessary objects are in place.

While the roles in this example support removing the applications (by setting state=absent), this is not necessary for roles in Ansible Operators. When the Ansible Operator creates Kubernetes objects, it injects ownerReference fields into all created objects. As such, deleting the CR (Gogs in this example) deletes all associated objects.

##### Build Operator
Before you can build with Docker you need to install, enable, and start the Docker daemon on your VM:
```
yum -y install docker
systemctl enable docker
systemctl start docker
```

**Build your Operator container image:**

In order to build and push the container image to a registry, you need a user ID at that registry—either Docker Hub or Quay.
For the purposes of this lab, it is recommended that you register a user ID at https://quay.io if you do not already have one.

The create account link is a bit hidden. Click on Sign In in the top right corner to see the login screen - which also has the link to Create Account.
Once you have registered a user ID, create a public repository in Quay called gogs-operator.

Make sure to select the public radio button on the Create New Repository page.

This is where you push your container image.
- Log in to Quay
```
export QUAY_ID=<your quay id>
docker login -u ${QUAY_ID} quay.io
```

- Use the Operator SDK to build the container image and tag it as quay.io/<your quay id>/gogs-operator:v0.0.1, making sure to replace <your quay id> with your Quay.io user ID:
```
cd $HOME/gogs-operator
operator-sdk build quay.io/${QUAY_ID}/gogs-operator:v0.0.1
```

- Push the container image to Quay:
```
docker push quay.io/${QUAY_ID}/gogs-operator:v0.0.1
```

Modify the Operator deployment definition (deploy/operator.yaml) by opening it up in a text editor.
- To use the image that you just pushed change the placeholder called {{ REPLACE_IMAGE }} and replace it with the actual image definition making sure to replace <your quay id> with your Quay.io user ID
- Set imagePullPolicy to IfNotPresent.
- Both changes have to be made twice because the image is used twice in the Operator pod.

##### Deploy Gogs Operator
Create the CRD for your Operator:
```
oc create -f ./deploy/crds/gpte_v1alpha1_gogs_crd.yaml
```

Create the `gogs-admin-rules` ClusterRole
```
echo '---
apiVersion: authorization.openshift.io/v1
kind: ClusterRole
metadata:
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
  name: gogs-admin-rules
rules:
- apiGroups:
   - apps
  resources:
  - deployments/finalizers
  verbs:
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - gpte.opentlc.com
  resources:
  - gogs
  - gogs/status
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch' | oc create -f -
```
- The label `rbac.authorization.k8s.io/aggregate-to-admin="true"` modifies the default project `admin` role to automatically grant this cluster role to any project owner, who then automatically gets `admin` permission on the project.

##### Deploy Operator
Operators can be deployed on a cluster-wide basis or on a namespace/project basis. In this lab, you deploy your Operator in your own project.

1) Log in to cluster
2) Create the Operator project
```
oc new-project gogs-operator --display-name="Gogs"
```
3) Deploy the artifacts one by one in the `deploy` subdirectory, starting with the `service_account` that your Operator will use:
```
oc create -f ./deploy/service_account.yaml
```

4) Create the role and RoleBinding that grant the correct permissions to the Operator using the following tips:

- The Operator SDK creates a skeleton role file for you. However, depending on what the Operator needs to create or manipulate, you likely need to customize the role.

- The Gogs Operator deploys some OpenShift-specific objects like a `Route`. `Routes` are defined in the `route.openshift.io` API group. So routes needs to be added to the list of resources that the Operator needs permissions for.

  - Edit the role definition `(vim ./deploy/role.yaml)` to include `routes` as an object that the Operator needs access to.
  - Add another apiGroup for `route.openshift.io`.

- You need to remove `events` because you do not have permission to manipulate events.

- Optionally, you can remove the entire `apps` API group because this Operator never creates any of these objects `(deployment, daemonset, statefulset, replicaset)`. But leave the second occurrance that grants permissions to `deployments/finalizers`.

- Replace the `"*"` resource in the `ocp4.freebsd.us` API group with `gogs` and `gogs/status`.

- Unlike basic Kubernetes, the OpenShift security settings require the verbs to be listed rather than using a `'*'` placeholder. So you need to add the `create, update, delete, get, list, watch, patch` verbs to the "", `route.openshift.io` and `ocp4.freebsd.us` API groups.

- Operator metering is not installed on this cluster, which means that you need to remove the `monitoring.coreos.com` API group.

Expect the final file to look like this:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gogs-operator
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - configmaps
  - secrets
  verbs:
  - create
  - update
  - delete
  - get
  - list
  - watch
  - patch
- apiGroups:
  - route.openshift.io
  resources:
  - routes
  verbs:
  - create
  - update
  - delete
  - get
  - list
  - watch
  - patch
- apiGroups:
  - apps
  resources:
  - deployments
  - deployments/finalizers
  verbs:
  - get
  - update
  - delete
  - get
  - list
  - watch
  - patch
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - apps
  resourceNames:
  - gogs-operator
  resources:
  - deployments/finalizers
  verbs:
  - update
- apiGroups:
  - gpte.opentlc.com
  resources:
  - gogs
  - gogs/status
  verbs:
  - create
  - update
  - delete
  - get
  - list
  - watch
  - patch
```

5) Create the role and RoleBinding (thereby granting the permissions in the role to the service account that the Operator uses):
```
oc create -f ./deploy/role.yaml
oc create -f ./deploy/role_binding.yaml

```

6) Create the deployment that points to your Operator image:
```
oc create -f ./deploy/operator.yaml
```

##### Use Operator to Create Gogs Server
To use an Operator to deploy an application, you need to create a custom resource, which is a Kubernetes object. When the custom resource is created in a project that the Operator is watching, the Operator creates the objects that make up that application based on the configuration options in the custom resource.

The Gogs Operator understands the following Configuration Options:

Name | Default | Required
postgresqlVolumeSize | 1Gi | No
gogsVolumeSize | 1Gi | No
gogsSsl | False | No

| In Kubernetes the convention for variable names is to use CamelCase, while in Ansible it is snake_case. As a result, the custom resource needs the variable names to be in CamelCase and the Ansible Playbooks and roles expect them to be in snake_case. The Ansible Operator base image handles this translation automatically.

All created objects need to have unique names. Usually this is achieved by using the name of the custom resource as a prefix to any created object. The roles you are using are built to satisfy these requirements.

- Create a custom resource to deploy a Gogs server in your project:
```
echo "apiVersion: ocp4.freebsd.us/v1alpha1
kind: Gogs
metadata:
  name: gogs-server
spec:
  postgresqlVolumeSize: 4Gi
  gogsVolumeSize: 4Gi
  gogsSsl: True" > $HOME/gogs-operator/gogs-server.yaml
```

- Create the Gogs server:
```
oc create -f $HOME/gogs-operator/gogs-server.yaml
```

- In a terminal window, watch as the pods come up:
```
watch oc get pod
```

- In another terminal window, tail the logs of the Gogs Operator pod to follow along as the Ansible Playbook executes:
```
oc logs -c operator -f <pod>
```

- Note that the Gogs Operator executes the Ansible Playbook over and over to make sure that all objects are in fact still there. You can configure the frequency of runs by changing the reconciliation time for the Operator.

- Note that the Ansible Operator adds owner references to all created objects. You do not use the Operator to delete a deployed application—instead, you delete the custom resource.

You can get the actual Ansible output from the playbook or role by examining the logs of the ansible container in the pod.
```
oc logs -c ansible -f <pod>
```

- Retrieve a list of Gogs objects in your project:
```
oc get gogs
```

- Examine the custom resource
```
oc describe gogs gogs-server
```

- Recall that the gogsSsl parameter in the custom resource was set to true. This means that the Operator set up a secure route for the Gogs server.
- Use the route in your web browser to access the home page of your Gogs server, which is already fully configured.
- Note that if you did not prefix the route with https://, the route is configured to automatically redirect insecure requests to the secure endpoint.
- If prompted, accept the insecure certificate if your cluster uses self-signed certificates.
- Deploy a second copy of the Gogs server by creating another custom resource called another-gogs and this time, set SSL to false:
```
echo "apiVersion: ocp4.freebsd.us/v1alpha1
kind: Gogs
metadata:
  name: another-gogs
spec:
  postgresqlVolumeSize: 4Gi
  gogsVolumeSize: 4Gi
  gogsSsl: False" > $HOME/gogs-operator/gogs-server-2.yaml

oc create -f $HOME/gogs-operator/gogs-server-2.yaml
```

##### Delete Gogs Servers
Deleting an object created by an Operator is as easy as deleting the custom resource. Operators that follow best practices set an `ownerReference` field in all created objects to enable cascading deletions. The Ansible Operator watches for Kubernetes API calls to create resources and adds the `ownerReference` field to every resource after it is successfully created. In this section, you examine the owner reference of the created pods.

- Note that the `ownerReference` points to an object of Gogs kind that has the name `gogs-server`. This `gogs-server` object is the custom resource that you created.

```
oc get gogs gogs-server -o yaml
```

- The Gogs object is the top-level object and therefore it does not contain another ownerReference.
- Secrets, PVCs, ConfigMaps, services, and routes also have the same ownerReference set.

##### Delete Gogs Operator
- Delete the Gogs Operator by deleting the deployment that hosts the Operator, the role, the RoleBinding, and the service account:
```
oc delete deployment gogs-operator
oc delete rolebinding gogs-operator
oc delete role gogs-operator
oc delete sa gogs-operator
```
