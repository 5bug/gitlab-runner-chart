# GitLab Runner Helm Chart

This chart deploys a GitLab Runner instance into your Kubernetes cluster. For more information, please review [our documentation](http://docs.gitlab.com/ee/install/kubernetes/gitlab_runner_chart.html).

官方chart仓库:https://gitlab.com/gitlab-org/charts/gitlab-runner

## 支持挂载宿主机目录
若gitlab-runner运行在Kubernetes环境里，官方默认的chart模板不支持挂载宿主机目录，但可以按照如下改动来支持。修改templates/configmap.yaml文件，在"# Start the runner"前面增加如下代码：
```bash
    cat >>/home/gitlab-runner/.gitlab-runner/config.toml <<EOF
        [[runners.kubernetes.volumes.host_path]]
          name = "docker-sock"
          mount_path = "/var/run/docker.sock"
          read_only = true
          host_path = "/var/run/docker.sock"        
        [[runners.kubernetes.volumes.host_path]]
          name = "cache"
          mount_path = "/cache"
          read_only = false
          host_path = "/data/gitlab-runner/cache" 
    EOF
```
根据自己需要挂载的目录自行调整挂载的目录。
## 使用方法
在helm3下部署gitlab-runner方法为：
```bash
helm package .
helm list | grep gitlab-runner > /dev/null
if [ $? -eq 0 ]; then
    helm upgrade --namespace default gitlab-runner *.tgz
else
    helm del --purge gitlab-runner
    helm install --namespace default --name gitlab-runner *.tgz    
fi
```

个人博客：http://www.5bug.wang 欢迎交流！
