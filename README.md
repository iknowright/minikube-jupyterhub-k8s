# Jupyterhub with Minikube

Template to run jupyterhub with minikube (with LDAP authenticator).

In this template, we are going to create 2 users, one is administrator (*admin_user*) another is just a normal user (*allowed_users*).

## Prerequisite
On your system, you should have the following setup:
* Minikube setup
* Helm

## Setup LDAP
By using this template, just run

`kubectl create -f ldap.yml`

to start LDAP in your cluster.

LDAP is the authenticator for our Jupterhub, in order to bind users information to Jupyterhub, we have to add entries (*organization, group, usernames*) to LDAP.

Here we provide a `ldap_config.txt` that specifies 2 users' information. Import it to your LDAP admin. Using `kubectl port-forward service/jupyterhub-ldap-admin 8081:80` to port LDAP admin to your `localhost:8081`.

Remember these 2 users will be then used in Jupyter setup section, and one of user will be the admin.

## Jupyterhub Setup
### Jupyterhub users setup
In `config.yml`, change *admin_user* and *allowed_users* to ones generated to LDAP. For instance, `admin_user: user1, allowed_users: user2`.

### Bind LDAP server address
Before we start our Jupyterhub service, we need to bind our LDAP authenticator. You can check LDAP IP by `kubectl describe pod jupyterhub-ldap`. Then fill in the proper IP inside `config.yml`.
```
LDAPAuthenticator:
    use_ssl: false
    server_address: {your_server_address}
```

### Run Jupterhub
In your terminal, first export two variable:
```
RELEASE=jhub
RELEASE=jhub
```

Then run:
`helm upgrade --cleanup-on-fail $RELEASE jupyterhub/jupyterhub --namespace $NAMESPACE --version=0.11.1-n121.h56e5ee17 --values config.yaml `

Now your Jupyterhub will run after couple of seconds.

We are not done yet, because we are using minikube for our Jupterhub, so we still need to port-forward the hub to localhost by running `kubectl port-forward service/proxy-public 8080:80 -n jhub`

Finally, you can start using the hub in `http://localhost:8080`.

## Reference
- [Zero to JupyterHub with Kubernetes](
https://zero-to-jupyterhub.readthedocs.io/en/stable/)

- [jupyterhub-kubernetes-ldap](https://extrapolations.dev/blog/2016/09/jupyterhub-kubernetes-ldap/)
