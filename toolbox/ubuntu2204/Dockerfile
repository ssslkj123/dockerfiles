# 1.基础镜像并配置国内加速源
FROM ubuntu:22.04
LABEL maintainer="shuanglulee@foxmail.com" \
      version="ubuntu2204 1.0.0" \
      description="Kubernetes Debug Toolchain Image (ubuntu2204 Base)"
# 处理ubuntu apt-get install过程当中的交互问题，ARG为构建镜像期间有效
ARG DEBIAN_FRONTEND=noninteractive

# 2.替换阿里云镜像源
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

# 3.安装核心工具链
RUN apt-get update && apt-get install -y --no-install-recommends \
    # ubuntu 包管理
    apt-cache aptitude apt-file \
    # 基础运维工具
    curl wget net-tools iproute2 tcpdump lsof procps vim tmux rsync linux-tools-generic  \
    # 网络诊断工具
    dnsutils iputils-ping telnet mtr-tiny nmap socat netcat-openbsd iftop iotop dnsutils iperf3 netperf  \
    # 系统调试工具
    strace ltrace gdb lshw psmisc htop jq git \
    # 编译工具链
    build-essential cmake autoconf libtool pkg-config libssl-dev zlib-dev zlib1g-dev automake libpcre3-dev libffi-dev gcc openssl-dev  \
    # 自动补全依赖
    bash-completion ca-certificates sudo file \
    # 清理缓存
    && rm -rf /var/lib/apt/lists/*

# 3.安装 Golang 1.22.0
RUN wget -O go1.22.0.linux-amd64.tar.gz https://dl.google.com/go/go1.22.0.linux-amd64.tar.gz \
    && tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz \
    && rm go1.22.0.linux-amd64.tar.gz
ENV PATH=$PATH:/usr/local/go/bin

# 4.安装 Python 3.12.0 
RUN wget https://www.python.org/ftp/python/3.12.0/Python-3.12.0.tgz \
    && tar -xzf Python-3.12.0.tgz \
    && cd Python-3.12.0 \
    && ./configure --enable-optimizations \
    && make -j $(nproc) \
    && make altinstall \
    && ln -sf /usr/local/bin/python3.12 /usr/bin/python3
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 5.安装 kubectl 并配置补全
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl && mv kubectl /usr/local/bin/ \
    && mkdir -p /etc/bash_completion.d \
    && kubectl completion bash > /etc/bash_completion.d/kubectl

# 6.安装 Helm 并配置补全
RUN curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash \
    && helm completion bash > /etc/bash_completion.d/helm

# 创建非 root 用户并切换
#RUN useradd -m debugger && echo "debugger ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
#USER debugger
#WORKDIR /home/debugger


# 7.加载补全配置
RUN echo "source /etc/bash_completion" >> ~/.bashrc
CMD ["/bin/bash", "-c", "trap : TERM INT; sleep infinity & wait"]
