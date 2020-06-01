# GitLab Runner Helm Chart

This chart deploys a GitLab Runner instance into your Kubernetes cluster. For more information, please review [our documentation](http://docs.gitlab.com/ee/install/kubernetes/gitlab_runner_chart.html).

官方chart仓库:https://gitlab.com/gitlab-org/charts/gitlab-runner

## 支持挂载宿主机目录和secret
若gitlab-runner运行在Kubernetes环境里，官方默认的chart模板不支持挂载宿主机目录，但可以按照如下改动来支持。修改templates/configmap.yaml文件，在"# Start the runner"前面增加如下代码：
```bash
   cat >>/home/gitlab-runner/.gitlab-runner/config.toml <<EOF
        {{- if .Values.hostMountPaths -}}
        {{ range .Values.hostMountPaths }}
        [[runners.kubernetes.volumes.host_path]]
          name = "{{ .name }}"
          read_only = {{ default true .readOnly }}
          mount_path = "{{ .mountPath }}"
          host_path = "{{ .hostPath }}"
        {{- end }}
        {{- end }}

        {{- if .Values.pvcMountPaths -}}
        {{ range .Values.pvcMountPaths }}
        [[runners.kubernetes.volumes.pvc]]
          name = "{{ .pvcName }}"
          mount_path = "{{ .mountPath }}"
        {{- end }}
        {{- end }}

        {{- if .Values.secretMountPaths -}}
        {{ range .Values.secretMountPaths }}
        [[runners.kubernetes.volumes.secret]]
          name = "{{ .secretName }}"
          mount_path = "{{ .mountPath }}"
          [runners.kubernetes.volumes.secret.items]
          {{- range .items }}
            {{ range $key, $val := . }}
              "{{ $key }}" = "{{ $val }}"
            {{- end }}
          {{- end }}
        {{- end }}
        {{- end }}
    EOF
```
根据自己需要挂载的目录自行调整挂载的目录，调整方法在values.yaml文件里增加指定相应的挂载参数，例如：
```yaml
# 挂载pvc存储
# pvcMountPaths:
#   - pvcName: "xxx"
#     mountPath: "/data/go/pkg/mod/"

# 挂载宿主机目录
hostMountPaths:
  - name: "docker-sock"
    readOnly: true
    mountPath: "/var/run/docker.sock"
    hostPath: "/var/run/docker.sock"
  - name: "docker-bin"
    readOnly: true
    mountPath: "/usr/bin/docker"
    hostPath: "/bin/docker"
  - name: "docker-so"
    readOnly: true
    mountPath: "/usr/lib64/libltdl.so.7"
    hostPath: "/usr/lib64/libltdl.so.7"
  - name: "log"
    readOnly: false
    mountPath: "/data/logs"
    hostPath: "/data/logs"

# 挂载docker的secret
# secretMountPaths:
#   - secretName: "docker-secret"
#     mountPath: "/root/.docker/"
#     items:
#       - docker-account: "config.json"
```


## 使用方法
在helm3下部署gitlab-runner方法为：
# install
```bash
helm install gitlab-runner -f values.yaml .
```
# update
```bash
helm upgrade gitlab-runner -f values.yaml .
```

个人博客：http://www.5bug.wang 欢迎交流！