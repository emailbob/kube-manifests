# Jenkins

For this example I'm putting everything in a namespace called `ci`

Create the namespace

`kubectl create namespace ci`

Start off by creating a Persistent Volume Claim with a size of 25Gi which will be mounted to /var/jenkins_home

`kubectl -f jenkins-pvc.yaml apply`

Create a ConfigMap with a plugins.txt file. Update this file with your choice of plugins. This will be mounted to /var/jenkins_config

`kubectl -f jenkins-cm.yaml apply`

The deployment will first launch an `initContainers` which will run `install-plugins.sh` against the plugins.txt file from the ConfigMap to install your plugins.  There is a `lifecycle` section commented out which you can also use to pre install the plugins.  If you choose to use this method make sure you comment out the initContainers section.  Warning with `PostStart` there are no guarantee that the install plugins script will be completed before the Jenkins entrypoint is launched. This will depend on how many plugins you have to install.

`kubectl -f jenkins-deploy.yaml apply`

Later on when you first access Jenkins it will ask you for an administrator password in `/var/jenkins_home/secrets/initialAdminPassword`. 

First get the name of the pod

`kubectl get pods -n ci`

```NAME                             READY     STATUS    RESTARTS   AGE
jenkins-leader-79f964547-s8lmc   1/1       Running   0          23h```

Once you have the name of the pod you can cat the contents of the file

`kubectl exec jenkins-leader-79f964547-s8lmc \
        -- cat /var/jenkins_home/secrets/initialAdminPassword`

Create Jenkins service to add the Jenkins pod to an AWS ELB

`kubectl -f jenkins-svc.yaml apply`

Run this to see to get your `LoadBalancer Ingress` so you can create a DNS CNAME entry for it

`kubectl describe svc jenkins-leader -n ci`