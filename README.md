# NodsJS-web-app-with-Kubernetes-and-Helm

A Helm chart i created that is used in the "K8s Helm Azure Devops app pipeline" **<a href="https://github.com/sincros121/K8s-Helm-Azure-Devops-app-pipeline.git" title="">repository</a>** to deploy the weight tracker web application.<br>
It includes the following manifests:
1. deployment for the web app 
2. deployment of statefulSet for the postgreSQL database
3. services to reach both deployments
4. config map and secrets
5. An ingress that uses the default ingressClass that is being installed by the NGINX ingress controller
6. A job that initializes the database

Please note all resources require a values.yaml file that is passed on to them via Helms `-f` flag to replace placeholder values.<br>

---
### NGINX ingress controller
The ingress controller is installed in a separate release before this one in order to obtain the public IP it creates to pass on to OKTA and the environment variables of the web-app containers.<br>
The command below to install the controller:<br>
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install nginx-controller ingress-nginx/ingress-nginx
```
A `--set` argument can be passed to set the ingressClass it creates as the default one for all classless ingress resources created:
`--set controller.setAsDefaultIngress=true`

---

### Notes and reflections:
The main issue i faced when trying to deploy a database is the initialization of the database,<br>
to ensure the attempt to initialize the database was done only when the DB container is ready, I created a job resource of the web-app the runs the command "npm run initdb",<br>
To further ensure it runs only when the DB is ready, it uses helm hooks: `"helm.sh/hook": post-install, post-upgrade` to only run post completed install/upgrade of the other resources.<br>
The last step to ensure it, was a postgres `initContainer` that ensures via command `pg_isready` towards our database service that it is up and running,<br>
the `npm run initdb` container will not run before the `initContainer` completes it's job successfully.
