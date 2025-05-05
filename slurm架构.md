root@trx:/home/ydf# docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS         PORTS      NAMES

10f3d1e77b5e   slurm-cluster:latest   "/entrypoint.sh"         4 hours ago   Up 4 hours                slurmd1

69b8dd4d667a   slurm-cluster:latest   "/entrypoint.sh"         4 hours ago   Up 4 hours                slurmd2

2310de17dbd1   slurm-cluster:latest   "/entrypoint.sh"         4 hours ago   Up 4 hours                slurmctld

d92ff1e5f870   slurm-cluster:latest   "/entrypoint.sh"         4 hours ago   Up 9 minutes              slurmdb

bcb2ee8b124f   mariadb:10.5           "docker-entrypoint.s…"   4 hours ago   Up 9 minutes   3306/tcp   mariadb


🧱 集群容器结构说明
容器名	镜像	作用	角色类型

slurmctld	slurm-cluster:latest	Slurm 控制节点，调度和管理作业	控制节点

slurmd1	slurm-cluster:latest	计算节点 1，实际运行作业	计算节点

slurmd2	slurm-cluster:latest	计算节点 2，实际运行作业	计算节点

slurmdb	slurm-cluster:latest	运行 slurmdbd（数据库守护进程）	账户记录

mariadb	mariadb:10.5	运行 MariaDB 数据库，存储作业数据	数据库


🔁 各组件之间的逻辑关系


![在线图片](https://github.com/yangdanfeng115/slurm/blob/main/slurm.png)


🔧 各组件的职责详细说明：
🟢 slurmctld（控制节点）
管理和调度作业（srun/sbatch 命令由它调度）

读取 slurm.conf，与所有节点通信

连接到 slurmdbd（用于记录作业）

🟢 slurmd1 / slurmd2（计算节点）
执行实际的作业（比如 sleep, python, mpi 等任务）

接受 slurmctld 派发的作业任务

运行 slurmd 守护进程

🟡 slurmdb（Slurm 数据库守护进程）
运行 slurmdbd，是 Slurm 的 作业会计信息管理系统

负责将作业数据写入数据库（如 job_table）

需要在 slurm.conf 和 slurmdbd.conf 中正确配置通信

🔵 mariadb（数据库服务）
用于保存作业记录、用户、账户、QOS 等信息

被 slurmdbd 使用，不能直接被 Slurm 其他组件访问

✅ 检查建议
如果你想确认这个集群是否已配置完好，可以检查：

slurmctld 能否 ping 到 slurmd1, slurmd2

sinfo
scontrol show nodes
slurmctld 与 slurmdbd 是否联通（检查 slurm.conf 中的 AccountingStorageHost）

slurmdbd 能否连接 mariadb（检查 slurmdbd.log）

各容器是否有互通网络（它们应该在同一 Docker 网络）

🧪 想要验证集群工作正常，可以做什么？
使用 srun 运行任务：


srun -N1 -n1 hostname
使用 sacct 查看作业记录是否写入数据库

使用 sinfo 查看节点状态是否都是 idle 或 alloc
