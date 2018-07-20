1. 创建Nginx Pod过程中报如下错误：
    #kubectlcreate -f nginx-pod.yaml
Error from server: error when creating "nginx-pod.yaml": Pod "nginx" is forbidden: no API token found for service account default/default, retry after the token is automatically created and added to the service account
   解决方法：
   1> 修改/etc/kubernetes/apiserver文件中KUBE_ADMISSION_CONTROL参数。
   修改前：
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
   去掉“ServiceAccount”选项。
