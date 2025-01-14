# containerd 配置文件说明

此文档提供了 `containerd` 配置文件 `config.toml` 的详细说明。`containerd` 是一个高性能的容器管理守护进程，负责管理容器的生命周期。

## 配置项说明

### 全局配置

- `disabled_plugins`: 禁用的插件列表。
- `imports`: 导入的配置文件列表。
- `oom_score`: OOM（内存不足）评分。
- `plugin_dir`: 插件目录。
- `required_plugins`: 必需的插件列表。
- `root`: 容器数据的根目录。
- `state`: 容器运行时的状态目录。
- `temp`: 临时文件目录。
- `version`: 配置文件版本。

### cgroup 配置

- `path`: cgroup 的路径。

### 调试配置

- `address`: 调试地址。
- `format`: 日志格式。
- `gid`: 组 ID。
- `level`: 日志级别。
- `uid`: 用户 ID。

### gRPC 配置

- `address`: gRPC 服务的地址。
- `gid`: 组 ID。
- `max_recv_message_size`: 最大接收消息大小。
- `max_send_message_size`: 最大发送消息大小。
- `tcp_address`: TCP 地址。
- `tcp_tls_ca`: TCP TLS CA 文件路径。
- `tcp_tls_cert`: TCP TLS 证书文件路径。
- `tcp_tls_key`: TCP TLS 密钥文件路径。
- `uid`: 用户 ID。

### 监控配置

- `address`: 监控服务的地址。
- `grpc_histogram`: 是否启用 gRPC 直方图。

### 插件配置

`containerd` 支持多种插件，以下是一些主要插件的配置项：

- **垃圾回收调度器** (`io.containerd.gc.v1.scheduler`):
  - `deletion_threshold`: 删除阈值。
  - `mutation_threshold`: 变更阈值。
  - `pause_threshold`: 暂停阈值。
  - `schedule_delay`: 调度延迟。
  - `startup_delay`: 启动延迟。

- **CRI 插件** (`io.containerd.grpc.v1.cri`):
  - `sandbox_image`: 沙箱镜像。
  - `stats_collect_period`: 统计收集周期。
  - `stream_idle_timeout`: 流闲置超时。

- **运行时配置** (`io.containerd.runtime.v1.linux`):
  - `runtime`: 运行时名称。
  - `shim`: shim 名称。

### 超时配置

- `io.containerd.timeout.bolt.open`: Bolt 数据库打开超时。
- `io.containerd.timeout.metrics.shimstats`: shim 统计超时。
- `io.containerd.timeout.shim.cleanup`: shim 清理超时。
- `io.containerd.timeout.shim.load`: shim 加载超时。
- `io.containerd.timeout.shim.shutdown`: shim 关闭超时。
- `io.containerd.timeout.task.state`: 任务状态超时。

### ttrpc 配置

- `address`: ttrpc 服务的地址。
- `gid`: 组 ID。
- `uid`: 用户 ID。

## 结论

此配置文件是 `containerd` 的核心部分，正确配置可以确保容器的高效管理和运行。请根据具体需求调整配置项。
