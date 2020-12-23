### Install Helm CLI Tool On k3s (example with nginx )

```
- master ip : 104.131.122.100
- worker1 ip :  104.236.15.11

```

1. check on github https://github.com/helm/helm/releases
 Linux amd64 (checksum / cacde7768420dd41111a4630e047c231afa01f67e49cc0c6429563e024da4b98)

```
root@node1-master:~# wget https://get.helm.sh/helm-v3.4.2-linux-amd64.tar.gz


root@node1-master:~# sha256sum helm-v3.4.2-linux-amd64.tar.gz
cacde7768420dd41111a4630e047c231afa01f67e49cc0c6429563e024da4b98  helm-v3.4.2-linux-amd64.tar.gz

root@node1-master:~# tar -xzvf helm-v3.4.2-linux-amd64.tar.gz

root@node1-master:~# mv linux-amd64/helm /usr/local/bin/helm
root@node1-master:~# helm version
version.BuildInfo{Version:"v3.4.2", GitCommit:"23dd3af5e19a02d4f4baa5b2f242645a1a3af629", GitTreeState:"clean", GoVersion:"go1.14.13"}

root@node1-master:~# helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories

root@node1-master:~# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

2.after install hel, you want to export this

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

### creae a package using helm, Github Pages example

1. create a repo on github called charts and create a new branch called  gh-pages

https://github.com/Pishoy/charts

2. set the branch  gh-pages as default , make sure enforce https on branch settings
3. now you can access your charts docs using https://pishoy.github.io/charts/ , message gonna appear as per index.html
4. clone repo , repo only contains index.html (optional)

```
root@node1-master:~# git clone git@github.com:Pishoy/charts.git
root@node1-master:~# ls charts
README.md   index.html
```

5. create a package from local helm charts ( to get local project clone it as below or use hel create buildachart )

```
root@node1-master:~# git clone git@github.com:Pishoy/charts.git -b main  buildachart
root@node1-master:~# cd buildachart/
root@node1-master:~/buildachart# ls
Chart.yaml  charts  templates  values.yaml
root@node1-master:~/buildachart# helm package .
Successfully packaged chart and saved it to: /root/buildachart/buildachart-0.1.0.tgz
root@node1-master:~/buildachart# ls
Chart.yaml  buildachart-0.1.0.tgz  charts  templates  values.yaml

root@node1-master:~/buildachart# mv  buildachart-0.1.0.tgz ~/charts

root@node1-master:~/buildachart# cd ..

root@node1-master:~#  helm repo index charts --url https://pishoy.github.io/charts/

root@node1-master:~# ls charts
README.md  buildachart-0.1.0.tgz  index.html  index.yaml

root@node1-master:~# cat charts/index.yaml
apiVersion: v1
entries:
  buildachart:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2020-12-23T13:12:02.690091922Z"
    description: A Helm chart for Kubernetes
    digest: d14cdb7e1e824647ced261b3d05db4e5ff6dad3f2862b7ca7de70a8ebc326f55
    name: buildachart
    type: application
    urls:
    - https://pishoy.github.io/charts/buildachart-0.1.0.tgz
    version: 0.1.0
generated: "2020-12-23T13:12:02.688684397Z"
```

6. list hel repo before add our repo
```
root@node1-master # helm repo list
NAME   	URL
stable 	https://charts.helm.sh/stable
bitnami	https://charts.bitnami.com/bitnami
```
7. add our repo
```
root@node1-master:~/charts# helm repo add  buildachart https://pishoy.github.io/charts/
"buildachart" has been added to your repositories

root@node1-master:~/charts# helm repo list
NAME       	URL
stable     	https://charts.helm.sh/stable
bitnami    	https://charts.bitnami.com/bitnami
buildachart	https://pishoy.github.io/charts/
```

8. list helm deployments, nothing deplyed
```
root@node1-master:~/charts# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```
9. deploy our app using added repo
```
root@node1-master:~/charts# helm install mytest buildachart/buildachart
NAME: mytest
LAST DEPLOYED: Wed Dec 23 13:23:48 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services cherry-chart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```
10. verify that deployment has been deployed
```
root@node1-master:~/charts# helm list
NAME  	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
mytest	default  	1       	2020-12-23 13:23:48.948944291 +0000 UTC	deployed	buildachart-0.1.0	1.16.0

root@node1-master:~/charts# kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
cherry-chart-bfbb59d64-nz4h9   1/1     Running   0          16s
```
11. now you can access nginx using http://104.131.122.100:31000 , as get from below
```
root@node1-master:~/buildachart#   export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services cherry-chart)
root@node1-master:~/buildachart#   export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
root@node1-master:~/buildachart#   echo http://$NODE_IP:$NODE_PORT
http://104.131.122.100:31000
```
