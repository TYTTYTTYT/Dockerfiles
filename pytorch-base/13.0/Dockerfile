FROM pytorch/pytorch:2.11.0-cuda13.0-cudnn9-devel
WORKDIR /workspace
ARG DEBIAN_FRONTEND=noninteractive
RUN apt update
RUN apt install htop -y
RUN apt install screen -y
RUN apt install wget -y
RUN apt install tar -y
RUN apt install zip  -y
RUN apt install unzip  -y
RUN apt install libpng-dev libjpeg-dev libopenexr-dev libtiff-dev libwebp-dev -y
RUN apt install ffmpeg -y
RUN apt install curl -y
RUN apt install iputils-ping -y
RUN apt install iproute2 -y
RUN apt install openssh-client -y
RUN apt install vim -y
RUN apt install nano -y
RUN apt install less -y
RUN apt install bsd-mailx -y
RUN apt install dnsutils -y
RUN apt install traceroute -y
RUN apt install net-tools -y
RUN apt install plocate -y
RUN apt install git -y
RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash && \
    apt-get install git-lfs -y && \
    git lfs install --system && \
    rm -rf /var/lib/apt/lists/*
# 1. Install certificates (ESSENTIAL for rclone to talk to cloud services)
RUN apt-get update && apt-get install -y ca-certificates fuse3 && rm -rf /var/lib/apt/lists/*
# 2. Copy rclone binary from the official image
COPY --from=rclone/rclone:latest /usr/local/bin/rclone /usr/local/bin/rclone
# 3. Verify installation
RUN rclone version
# Install Noto fonts for multilingual text rendering (Bengali, Tibetan, CJK, etc.)
RUN apt-get update && apt-get install -y fonts-noto-core fonts-noto-cjk fonts-noto-extra && \
    fc-cache -f -v && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y --no-install-recommends \
    texlive-full \
    latexmk \
    make \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
RUN cp /root/.bashrc /etc/skel/.bashrc 2>/dev/null; ln -sf /workspace/.bashrc /root/.bashrc
CMD cp -n /etc/skel/.bashrc /workspace/.bashrc 2>/dev/null; /bin/bash
