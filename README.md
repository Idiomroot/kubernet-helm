## k8s安装helm
一、二进制安装helm客户端
###### # git clone https://github.com/Idiomroot/kubernet-helm.git
###### # cd /home/tools/ && wget https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz
###### # tar -xf helm-v2.14.3-linux-amd64.tar.gz  && mv linux-amd64/helm /usr/local/bin/helm
###### # helm  help  (运行成功则表明helm客户端安装成功)
二、安装服务端tiller
###### # yum -y install socat
###### # kubectl create serviceaccount --namespace kube-system tiller
###### # kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
###### # helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
###### # kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
###### # kubectl -n kube-system get pods|grep tiller
tiller-deploy-78d6566545-7fx9p          1/1     Running   0          6s
###### # helm version       
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
注意，client和server端的版本要一致，不然安装服务会报错。
三、测试用helm安装服务
###### # helm search nginx
###### NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                                 
###### stable/nginx-ingress    0.9.5           0.10.2          An nginx Ingress controller that uses ConfigMap to store ...
###### stable/nginx-lego       0.3.1                           Chart for nginx-ingress-controller and kube-lego            
###### stable/gcloud-endpoints 0.1.0                           Develop, deploy, protect and monitor your APIs with Googl...
###### # helm install stable/nginx-ingress --name test
###### NAME:   test
###### LAST DEPLOYED: Fri Aug  9 06:43:25 2019
###### NAMESPACE: default
###### STATUS: DEPLOYED

###### RESOURCES:
###### ==> v1/ConfigMap
###### NAME                           DATA  AGE
###### test-nginx-ingress-controller  1     0s

###### ==> v1/Pod(related)
###### NAME                                                 READY  STATUS             RESTARTS  AGE
###### test-nginx-ingress-controller-6d8b7645c8-d8l9d       0/1    ContainerCreating  0         0s
###### test-nginx-ingress-default-backend-5674776778-5bc9w  0/1    ContainerCreating  0         0s

###### ==> v1/Service
###### NAME                                TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)                     AGE
###### test-nginx-ingress-controller       LoadBalancer  10.254.65.36  <pending>    80:31557/TCP,443:31836/TCP  0s
###### test-nginx-ingress-default-backend  ClusterIP     10.254.48.28  <none>       80/TCP                      0s

###### ==> v1beta1/Deployment
###### NAME                                READY  UP-TO-DATE  AVAILABLE  AGE
###### test-nginx-ingress-controller       0/1    1           0          0s
###### test-nginx-ingress-default-backend  0/1    1           0          0s

###### ==> v1beta1/PodDisruptionBudget
###### NAME                                MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
###### test-nginx-ingress-controller       1              N/A              0                    0s
###### test-nginx-ingress-default-backend  1              N/A              0                    0s

###### NOTES:
###### The nginx-ingress controller has been installed.
###### It may take a few minutes for the LoadBalancer IP to be available.
###### You can watch the status by running 'kubectl --namespace default get services -o wide -w test-nginx-ingress-controller'

###### An example Ingress that makes use of the controller:

######   apiVersion: extensions/v1beta1
######   kind: Ingress
######   metadata:
######     annotations:
######       kubernetes.io/ingress.class: nginx
######     name: example
######     namespace: foo
######   spec:
######     rules:
######       - host: www.example.com
######         http:
######           paths:
######             - backend:
######                 serviceName: exampleService
######                 servicePort: 80
######               path: /
######     # This section is only required if TLS is to be enabled for the Ingress
######     tls:
######         - hosts:
######             - www.example.com
######           secretName: example-tls

###### If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

######   apiVersion: v1
######   kind: Secret
######   metadata:
######     name: example-tls
######     namespace: foo
######   data:
######     tls.crt: <base64 encoded cert>
######     tls.key: <base64 encoded key>
######   type: kubernetes.io/tls

###### # helm list
NAME    REVISION        UPDATED                         STATUS          CHART                   APP VERSION     NAMESPACE
test    1               Fri Aug  9 06:43:25 2019        DEPLOYED        nginx-ingress-0.9.5     0.10.2          default  
删除安装的release
###### # helm  delete test   --purge 
release "test" deleted

四、创建自己的chart
###### # cd /root/ && helm create nginx
###### # tree nginx         
###### nginx
###### ├── charts
###### ├── Chart.yaml
###### ├── templates
###### │   ├── deployment.yaml
###### │   ├── _helpers.tpl
###### │   ├── ingress.yaml
###### │   ├── NOTES.txt
###### │   ├── service.yaml
###### │   └── tests
###### │       └── test-connection.yaml
###### └── values.yaml

###### 3 directories, 8 files
###### # helm package nginx
###### Successfully packaged chart and saved it to: /root/nginx-0.1.0.tgz
###### # helm search nginx
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                                 
###### local/nginx             0.1.0           1.0             A Helm chart for Kubernetes                                 
###### stable/nginx-ingress    0.9.5           0.10.2          An nginx Ingress controller that uses ConfigMap to store ...
###### stable/nginx-lego       0.3.1                           Chart for nginx-ingress-controller and kube-lego            
###### stable/gcloud-endpoints 0.1.0                           Develop, deploy, protect and monitor your APIs with Googl..
###### # helm  serve
###### Regenerating index. This may take a moment.
###### Now serving you on 127.0.0.1:8879
###### 安装chart可以通过HTTP server的方式提供，访问本机的8879端口。
###### # curl http://127.0.0.1:8879
###### <html>
 <head>
         <title>Helm Repository</title>
 </head>
 <h1>Helm Charts Repository</h1>
<ul>

   <li>nginx<ul>
     <li><a href="http://127.0.0.1:8879/nginx-0.1.0.tgz">nginx-0.1.0</a></li>
   </ul>
   </li>

 </ul>
 <body>
 <p>Last Generated: 2019-08-09 07:04:48.034892153 &#43;0800 CST</p>
 </body>
 </html>
注释：
如果要使用第三方chat库，则需要添加
比如添加fabric8库：helm repo add fabric8 https://fabric8.io/helm
