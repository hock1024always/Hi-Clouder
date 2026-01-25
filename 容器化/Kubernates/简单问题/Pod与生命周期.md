# 简述Kubernetes中什么是静态Pod？

**思路**：

- 定义：由 kubelet 直接管理、不通过 API Server 创建的 Pod；
- 配置方式：通过本地文件或 HTTP 源；
- 特点：仅存在于单个节点，无法被控制器（如 Deployment）管理；
- 典型用途：运行控制平面组件（如 kube-apiserver）。

**回答**：

静态 Pod 是由 **kubelet 直接从本地配置文件（或 HTTP 端点）读取并创建**的 Pod，不经过 API Server，因此不会在 etcd 中持久化。它仅存在于该节点上，无法被 Deployment、DaemonSet 等控制器管理。Kubernetes 集群的控制平面组件（如 kube-apisvc、etcd）常以静态 Pod 方式部署，确保即使 API Server 不可用，这些关键组件仍能启动。

# 简述Kubernetes中Pod可能位于的状态？

#### **思路**：

列举官方定义的五种核心状态，简要说明每种状态的含义，避免混淆“容器状态”和“Pod 状态”。

#### **回答**：

Pod 的生命周期状态（`.status.phase`）主要有五种：

- **Pending**：Pod 已被创建，但容器尚未创建（如镜像拉取中、调度未完成）；
- **Running**：Pod 已绑定到节点，至少一个容器在运行或已启动；
- **Succeeded**：Pod 中所有容器成功终止且不再重启（用于 Job 场景）；
- **Failed**：Pod 中至少一个容器以非零状态退出，且重启策略不重试；
- **Unknown**：因通信问题（如节点失联）无法获取 Pod 状态。

> 注意：这不同于容器的 `Waiting/Running/Terminated` 状态。

# 简述Kubernetes创建一个Pod的主要流程？

**思路**：

按时间线梳理：用户提交 → API Server 校验存储 → 调度器分配节点 → kubelet 启动容器。

**回答**：

创建 Pod 的主要流程如下：

1. 用户通过 kubectl 提交 Pod YAML 到 **API Server**；
2. API Server 验证并持久化 Pod 对象到 **etcd**；
3. **kube-scheduler** watch 到未绑定节点的 Pod，根据调度算法（资源、亲和性等）选择合适节点，并将绑定信息写回 API Server；
4. 目标节点的 **kubelet** watch 到分配给自己的 Pod，调用容器运行时（如 containerd）拉取镜像并启动容器；
5. kubelet 持续上报 Pod 状态至 API Server，完成整个创建闭环。

# 简述Kubernetes中Pod的重启策略？

#### **思路**：

明确 `restartPolicy` 字段的三种取值及其适用场景，强调它作用于 Pod 内所有容器。

#### **回答**：

Pod 的重启策略由 `spec.restartPolicy` 定义，仅支持以下三种：

- **Always**（默认）：容器退出后自动重启，适用于长期运行的服务（如 Web 应用）；
- **OnFailure**：仅当容器以非零状态退出时才重启，适用于批处理任务；
- **Never**：从不重启，容器退出即终止，常用于调试或一次性任务。

> 注意：重启策略由 **kubelet** 在节点本地执行，与控制器（如 Deployment）无关。

# 简述Kubernetes Pod如何实现对节点的资源控制？

#### **思路**：

从容器资源请求（requests）和限制（limits）切入，说明如何通过 Cgroups 实现隔离。

#### **回答**：

Kubernetes 通过为 Pod 中每个容器设置 **CPU 和内存的 requests 与 limits**，由 kubelet 调用底层容器运行时（如 containerd）配置 **Linux Cgroups**，从而实现对节点资源的隔离与控制：

- **CPU**：通过 cpu.shares（requests）和 cpu.cfs_quota_us（limits）限制使用份额与上限；
- **内存**：通过 memory.limit_in_bytes 设置硬限制，超限会触发 OOM Kill。
  这样既保障了 Pod 的资源可用性，又防止其过度占用影响其他工作负载。



# 简述Kubernetes Requests和Limits如何影响Pod的调度？

#### **思路**：

区分调度阶段（只看 requests）和运行阶段（受 limits 限制），强调调度器依据的是 requests。

#### **回答**：

- **调度阶段**：kube-scheduler **仅依据容器的 `resources.requests`** 判断节点是否有足够资源。只有当节点可分配资源 ≥ Pod 所有容器 requests 总和时，才会将 Pod 调度到该节点；
- **运行阶段**：容器实际使用受 `resources.limits` 限制，超限会被内核限制（CPU 节流）或杀死（内存 OOM）。

> 因此，requests 决定“能否调度”，limits 决定“运行时是否被限制”。



# 简述Kubernetes初始化容器（init container）？

#### **思路**：

说明其顺序性、阻塞性、用途（前置条件准备），并与主容器对比。

#### **回答**：

Init Container 是在主应用容器启动**之前**按顺序运行的临时容器，具有以下特点：

- 必须**全部成功完成**后，主容器才会启动；
- 常用于执行前置任务，如等待依赖服务就绪、下载配置、执行数据库迁移等；
- 与主容器共享 Pod 的网络和存储卷，但拥有独立的镜像和命令；
- 若 Init Container 失败，kubelet 会按 Pod 重启策略重试（但通常用于一次性任务，故多设为 Never 或 OnFailure）。
  它是实现 Pod 启动前“准备就绪”逻辑的标准机制。
