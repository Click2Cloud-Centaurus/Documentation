
# Centaurus Portal
This design document is a proposal for enhancing the dashboard UI that
allows users to manage Centaurus Cluster, Tenants, Users, and
Quotas in an intuitive way.

## Goals 
Following are the added features in Centaurus portal(Dashboard UI):
* Manage Centaurus Cluster
* Manage multiple TPs and RPs using dashboard (by enabling client to fetch details from all TPs and RPs)
* Enable user to login using username and password (instead of token)
* Cluster management
* Tenant management
* User management
* Monitoring
* VM workload feature
* Managing:
  * Quota
  * Namespace
  * Roles
  * Cluster Role


## Background
Below operations can be obtained by the cluster admin and tenant admin using CLI (i.e. using `kubectl` utility) :
* Tenant partition detail
* Resource partition detail
* Tenant Management
* Cluster role and role CRUD operation
* User Management
* Quota CRUD operation
* Namespace CRUD operation
* Role CRUD operation
* Cluster Monitoring
* VM workload management

None of these are reflected in the current version of Dashboard UI. There should be a simplified, more user-friendly way to manage the cluster, tenants and users.

## Overview
### User Management
![](img-3.png)

### Cluster admin profile
Cluster admin can perform following operation using Dashboard UI:
* Create tenant along with tenant admin
* Delete tenant
* List tenant
* Create tenant admin
* Monitor health checks & resource utilization for each and every partition
* Create RBAC roles and role bindings for other fine-grained cluster admins

Following YAML is being used to create Cluster admin

```cgo
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-admin
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: cluster-admin
    namespace: default
    apiGroup: ""
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

![img.png](img.png)

### Tenant admin profile
Tenant admin can perform following operation using Dashboard UI:
* Creating other fine-grained tenant admins and regular tenant users
* Monitor health checks & resource utilization for its own respective tenant within the Centaurus cluster
* List/create/delete  tenant users
* Create RBAC roles and role bindings in the tenant
* Manage namespace quotas for a tenant

Following YAML is being used to create tenant admin
```json
apiVersion: v1
kind: Namespace
metadata:
  tenant: tenant-admin
  name: default
---
apiVersion: v1
kind: ServiceAccount
metadata:
  tenant: tenant-admin
  name: tenant-admin
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  tenant: tenant-admin
  name: tenant-admin
  namespace: default
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tenant-admin
  tenant: tenant-admin
  namespace: default
subjects:
  - kind: ServiceAccount
    name: tenant-admin
    namespace: default
    apiGroup: ""
roleRef:
  kind: ClusterRole
  name: tenant-admin
  apiGroup: rbac.authorization.k8s.io
```
![](img-1.png)

### Tenant user profile
Tenant user can perform following operation using Dashboard UI:
* Application deployment
* VM workload management
* Monitoring and resource utilization according to RBAC
* 
```bigquery
apiVersion: v1
kind: Namespace
metadata:
  name: user-namespace
  tenant: tenant-name
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tenant-user
  namespace: user-namespace
  tenant: tenant-name
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: tenant-user
  namespace: user-namespace
  tenant: tenant-name
subjects:
  - kind: ServiceAccount
    name: tenant-user
    namespace: user-namespace
    apiGroup: ""
roleRef:
  kind: Role
  name: tenant-user
  apiGroup: rbac.authorization.k8s.io
``

![img-2.png](img-2.png)

## Feature details
___
#### 1. IAM service details
IAM service is a service that manages users, roles, and permissions.
This service will be used to manage Centaurus user's username and password.
###### API added
* Create User(Cluster admin/Tenant admin/Tenant user)
* List all users
* Get details of specific user
* Delete user

#### 2. Tenant and tenant-admin Creation

At the time of a tenant creation by *Cluster admin*, a default tenant admin user will be created inside the newly created tenant. Once done, the default tenant admin can do everything inside the tenant without turning to cluster admin for any tenant management functions.

![](img-4.png)

#### 3. Tenant User Creation
For a tenant, a user can be created. Tenant user can work on specific namespace within the tenant.

#### 4. Cluster Monitoring
* Cluster admin can monitor health checks & resource utilization for each and every partition
* Tenant admin can monitor health checks & resource utilization for its own respective tenant within the Centaurus cluster
* Tenant user can monitor health checks & resource utilization according to RBAC

###### API developed in Dashboard backend
* To get details of Tenant Partition
* To get details of Resource Partition

### Dashboard detailed Design

##### 1. Login Page

![](img-5.png)

##### 2. Cluster Monitoring
* List of all the partitions available
###### Enable multi-config support in dashboard client:
In centaurus cluster, for 2TP and 2RPs cluster, user will have 4 configs. So dashboard's client can connect to respective API server(respective TP) in which that tenant is located.
For eg. if we have 2TPs and 2RPs cluster, then all tenants with prefix between `a` to `m` will get created in TP1 and tenants with prefix between `n` to `z` will get created in TP2.

![](img_4.png)

* Inside Resource Partition details, user will be able see the details of all nodes and resources

![](img_5.png)

* Inside Tenant Partition details, user will be able see the details of all the tenants.

![](img_6.png)


![](img_7.png)

##### 3. Tenant Monitoring
* It will show details of all resources within a tenant

![](img_8.png)

##### 4. Tenant Operation
***List Tenants***

![](img_9.png)
***Create Tenant Admin operation***

![](img_10.png)

##### 5. Managing Namespace
* List of all Namespaces created

![](img_11.png)

##### 6. Access Control
***Roles and Cluster roles***

![](img_12.png)


![](img_13.png)

##### 7. Managing Quotas
* List of quotas for a tenant

![](img_14.png)

* Tenant admin can manage quota for different namespaces within a tenant and also Tenant admin can update the quota assigned to a tenant

![](img_15.png)

##### 7. User Management

* List of all the users created

![](img_16.png)

* Create a new user

![](img_17.png)


### Developement Portal Link

***Link***: [Centaurus Portal](https://146.148.106.48:9443/#/login)

***Username***: `centaurus`

***Password***: `Centaurus@123`
