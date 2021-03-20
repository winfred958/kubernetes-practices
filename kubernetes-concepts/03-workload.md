# [workload](https://kubernetes.io/docs/concepts/workloads/)

- Pod & workload
    - 运行在 Kubernetes 上的 application 称为 workload.
    - 用户的 workload 运行在Pod中, Pod 代表集群中运行的1个Container Group(1个或多个Container).
    - Pod 有明确的生命周期 [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/), k8s 会根据用户的定义维护
      Pod 生命周期.

- Workload Resource & Pod
    - Workload Resource 用来创建和管理 Pods.
    - 为了简化使用, 用户不必直接控制每1个Pod, 而只需要定义 **Workload Resource** 管理一组 Pod.
    - 根据**Workload Resource**定义, [k8s controller](https://kubernetes.io/docs/concepts/architecture/controller/)
      会维护Pod正确的副本数, 匹配用户指定的状态.
    - Workload Resource 主要分类:
        - [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
          and [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
            - 部署无状态的application
            - ```text
              注意:
              大多数情况下使用 Deployment (推荐).
              ReplicaSet 只在用户自定义编排时使用.
              ```
        - [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
            - 让用户运行1个或多个相关的可以跟踪状态的Pod.
        - [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
            - 一般用来定义集群的基础设施.
            - Node 只能能存在**1个同类型的Pod**.
        - [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
          and [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
            - ```text
              运行完(completion)即停止(stop)的进程, Job一次性, CronJob 周期性.
              ```

## [Pod](https://kubernetes.io/docs/concepts/workloads/pods/)

```text
Pod 是 k8s 创建和可部署(调度)的最小单元.
1个Pod由是由1个或多个 Container 组成的 group.
Pod内的Container共享网络和存储资源(cgroup, namespace, 共享 filesystem volume).
```

- ### [Pod 的使用](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)
    - #### [Pod 如何管理多个 Container](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)

- ### [Working with Pods](https://kubernetes.io/docs/concepts/workloads/pods/#working-with-pods)
    - #### [Pods and controllers](https://kubernetes.io/docs/concepts/workloads/pods/#pods-and-controllers)
        - workload resource 管理 pod 的**例子**
            - [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)
            - [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)
            - [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)

    - #### [Pod templates](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates)

- ### [Pod 更新(update)和替换(replacement)](https://kubernetes.io/docs/concepts/workloads/pods/#pod-update-and-replacement)

- ### [资源共享和通信, Resource sharing and communication](https://kubernetes.io/docs/concepts/workloads/pods/#resource-sharing-and-communication)

## [Workload Resources](https://kubernetes.io/docs/concepts/workloads/controllers/)