### 1. 镜像准备
 * docker.io/prom/prometheus 
 * docker.io/somedayiamold/kube-state-metrics (因为本地部署的k8s少了service account管理，因此无法直接使用quay.io/coreos/kube-state-metrics:v1.3.0)，因为使用了自己编译的镜像，后面会详细讲docker.io/giantswarm/tiny-tools
 * docker.io/dockermuenster/caddy
 * docker.io/grafana/grafana
 * docker.io/prom/node-exporter
 
### 2. 创建namespace
 创建命令:
 kubectl create -f namespace.yaml

### 3. 创建node exporter
创建node-exporter daemonset和node-exporter service命令：  
kubectl create -f exporter-daemonset.yaml -f exporter-service.yaml  
### 4. 创建rbac  
kube-state-metrics默认使用kube-state-metrics账号，prometheus使用prometheus-k8s，后面两个服务会用到  
创建 rbac命令：  
kubectl create -f rbac.yaml  
### 5. 创建kube-state-metric  
本地部署的k8s没有打开serviceaccount控制，因此会导致kube-state-metric在容器内运行时报错，详见 https://github.com/kubernetes/kube-state-metrics/issues/398  
因此，我们需要自己打包一个镜像，把apiserver传给kube-state-metric  
利用dockerhub来创建镜像，参考 https://blog.csdn.net/qq_27028561/article/details/79064414  
创建kube-state-metrics deployment和kube-state-metrics service命令：  
kubectl create -f state-metrics-deployment.yaml -f state-metrics-service.yaml  
### 6. 创建node-directory-size-metrics  
创建node-directory-size-metrics daemonset命令：  
kubectl create -f node-directory-size-metrics-daemonset.yaml  
### 7. 创建prometheus configmap
因为我们使用的2.2版本的prometheus，因此configmap需要按照最新的配置来改，具体参考 https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml  
注意：如果使用k8s 1.7.3之后的版本，kubernetes-cadvisor是必须的，因为container_打头的监控项都已经从kubelet移到了cadvisor中，如果是1.6之前的版本，这段配置需要移除
暂时不使用报警功能，因此注释掉rule_files配置   
创建prometheus-core命令：  
kubectl create -f prometheus-core-deployment.yaml  
### 8. 创建prometheus service  
创建prometheus service命令：  
kubectl create -f prometheus-service.yaml  
### 9. 部署grafana
创建grafana命令：  
kubectl create -f grafana-service.yaml -f grafana-deployment.yaml  
导入prometheus数据到grafana  
创建configmap  
kubectl create -f grafana-dashboard-configmap.yaml  
执行导入job  
kubectl create -f grafana-dashboard-job.yaml  
执行成功之后打开grafana，会发现报错DS_PROMETHEUS找不到，在dashboard设置中添加一个变量DS_PROMETHEUS即可  
通过grafana可以导入prometheus的dashboard和其他k8s dashboard，自定义配置，具体参考grafana相关文档  

### 参考：  
* http://dockone.io/article/2579  
* https://blog.csdn.net/wenwst/article/details/76624019  
* https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml  
* https://github.com/giantswarm/kubernetes-prometheus/tree/master/manifests  
* http://blog.51cto.com/newfly/2061135  
* https://blog.csdn.net/zqg5258423/article/details/53119009  
* https://github.com/kubernetes/kube-state-metrics  
* https://blog.csdn.net/qq_27028561/article/details/79064414  
