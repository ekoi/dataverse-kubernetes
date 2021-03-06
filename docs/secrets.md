# Handling passwords with K8s Secrets
Please use [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) and *mount them as volumes*.
See also [here](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#create-a-pod-that-has-access-to-the-secret-data-through-a-volume).

Currently understood secrets in the container, mounted at `SECRETS_DIR=/opt/dataverse/secrets`:
1. `rserve/password` - optional, only needed when using a RServe server.
2. `doi/password` - needed when you use DOIs for PIDs.
3. `db/password` - required - guess why?
4. `api/key` - required because you want the *unblock-key* for anything serious.
5. `s3/access-key` and `s3/secret-key` - needed when you want to use S3 storage. See #28.

A [password alias](https://docs.oracle.com/cd/E19798-01/821-1751/ghgqc/index.html) is automatically created and used for those, no need to provide
those yourself. (see [default.config](https://github.com/IQSS/dataverse-kubernetes/blob/master/docker/dataverse-k8s/bin/default.config))

You can of course map other parts of the secret like usernames to an environment
variable like `doi_username` etc.

### Use a `Secret` to configure PostgreSQL connection details
You may use the *dataverse-postgres* secret above to configure database name,
database user and password without adding these details to the `ConfigMap`.

Customize the following example to use it:
```
kubectl create secret generic dataverse-postgresql \
            --from-literal=username='dataverse' \
            --from-literal=password='changeme' \
            --from-literal=database='mydataverse'
```
