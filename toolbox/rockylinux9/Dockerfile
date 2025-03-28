# 1.添加维护者元数据（LABEL 实现）
FROM rockylinux:9
LABEL maintainer="shuanglulee@foxmail.com" \
      version="1.0.0" \
      description="Kubernetes Debug Toolchain Image (RockyLinux 9 Base)"

# 2.安装epel
RUN dnf install -y --nodocs --setopt=install_weak_deps=0 \
    epel-release \
    && dnf clean all && rm -rf /var/cache/dnf
RUN cp /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup && sed -i 's|^metalink=|#metalink=|g' /etc/yum.repos.d/epel.repo && sed -i 's|^#baseurl=.*|baseurl=https://mirrors.aliyun.com/epel/9/Everything/x86_64/|g' /etc/yum.repos.d/epel.repo

RUN sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak /etc/yum.repos.d/rocky*.repo
# 3.安装工具链
RUN dnf install -y --nodocs --setopt=install_weak_deps=0 \
    # 基础工具
    curl wget net-tools iproute tcpdump lsof procps-ng vim-enhanced tmux iftop iotop rsync perf  \
    # 网络工具
    bind-utils iputils mtr nmap socat nc telnet bind-utils iperf3 netperf stress  \
    # 调试工具
    strace ltrace gdb psmisc htop jq git \
    # 自动补全依赖
    bash-completion ca-certificates sudo file \
    #编译工具
    gcc gcc-c++ make pcre-devel zlib-devel openssl-devel libffi-dev\
    && dnf clean all && rm -rf /var/cache/dnf

# 4.安装golang 1.24
RUN wget -O go1.24.1.linux-amd64.tar.gz https://go.dev/dl/go1.24.1.linux-amd64.tar.gz \
    && tar -C /usr/local -xzf go1.24.1.linux-amd64.tar.gz \
    && rm go1.24.1.linux-amd64.tar.gz
ENV PATH=$PATH:/usr/local/go/bin
# 镜像加速
ENV GOPROXY=https://goproxy.cn,direct

# 5.安装python 3.13
RUN wget https://www.python.org/ftp/python/3.13.2/Python-3.13.2.tgz \
    && tar -xzf Python-3.13.2.tgz \
    && cd Python-3.13.2 \
    && ./configure --enable-optimizations \
    && make -j $(nproc) \
    && make altinstall \
    && ln -sf /usr/local/bin/python3.13 /usr/bin/python3
# pip源加速
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple


# 6.安装 kubectl 并配置补全
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl && mv kubectl /usr/local/bin/ \
    && mkdir -p /etc/bash_completion.d \
    && kubectl completion bash > /etc/bash_completion.d/kubectl

# 7.安装 Helm 并配置补全
RUN curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash \
    && helm completion bash > /etc/bash_completion.d/helm

# 创建非 root 用户并切换
# RUN useradd -m debugger && echo "debugger ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
# USER debugger
# WORKDIR /home/debugger

# 8.加载补全配置
RUN echo "source /etc/bash_completion" >> ~/.bashrc

# 入口配置
CMD ["/bin/bash", "-c", "trap : TERM INT; sleep infinity & wait"]
