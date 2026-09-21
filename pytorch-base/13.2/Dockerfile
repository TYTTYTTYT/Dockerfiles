FROM pytorch/pytorch:2.14.0-cuda13.2-cudnn9-devel
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
# Common CLI / dev / paper-figure utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    tmux rsync jq tree ncdu lsof strace file bc \
    ripgrep fd-find \
    iotop nvtop \
    pigz p7zip-full \
    build-essential cmake pkg-config \
    graphviz imagemagick poppler-utils inkscape \
    aria2 zstd pv parallel btop iftop gh pandoc sqlite3 \
    ninja-build ccache gdb clang clang-format \
    && rm -rf /var/lib/apt/lists/*
# R (latest from CRAN apt repo) + system libs commonly needed to build R packages
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates gnupg && \
    curl -fsSL https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc \
        | gpg --dearmor -o /usr/share/keyrings/cran.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/cran.gpg] https://cloud.r-project.org/bin/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME)-cran40/" \
        > /etc/apt/sources.list.d/cran.list && \
    apt-get update && apt-get install -y --no-install-recommends \
    r-base r-base-dev \
    libcurl4-openssl-dev libssl-dev libxml2-dev \
    libfontconfig1-dev libfreetype6-dev libharfbuzz-dev libfribidi-dev \
    && rm -rf /var/lib/apt/lists/*
# renv (per-project environments) + pak (fast installer) from Posit Package Manager
RUN CODENAME=$(. /etc/os-release && echo $VERSION_CODENAME) && \
    Rscript -e "install.packages(c('renv', 'pak'), repos = 'https://packagemanager.posit.co/cran/__linux__/${CODENAME}/latest')" && \
    Rscript -e "library(renv); library(pak)"
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
# Generate all UTF-8 locales so SSH-forwarded LANG values (e.g. en_US.UTF-8) resolve correctly
RUN apt-get update && apt-get install -y locales && \
    sed -i 's/^# *\(.*\.UTF-8\)/\1/' /etc/locale.gen && \
    locale-gen && \
    rm -rf /var/lib/apt/lists/*
ENV LANG=en_US.UTF-8
ENV SHELL=/bin/bash
ENV CUDA_HOME=/usr/local/cuda
# Interactive-shell defaults for all users, appended to /etc/bash.bashrc.
# If $WORKSPACE (default /workspace; override with `docker run -e WORKSPACE=...`) is a writable
# directory, it becomes HOME, so ~/.bashrc, ~/.local, history and tmp files live on that volume.
ENV WORKSPACE=/workspace
RUN cat >> /etc/bash.bashrc <<'EOF'

# --- image defaults ---
force_color_prompt=yes
alias fd=fdfind
HISTCONTROL=ignoredups:ignorespace
export HISTSIZE=100000 HISTFILESIZE=200000
export HISTTIMEFORMAT="%F %T "
shopt -s histappend

if [ -n "$WORKSPACE" ] && [ -d "$WORKSPACE" ] && [ -w "$WORKSPACE" ]; then
    mkdir -p "$WORKSPACE/.local/bin" "$WORKSPACE/tmp"
    export HOME="$WORKSPACE"
    export HISTFILE="$HOME/.bash_history"
    export TMPDIR="$WORKSPACE/tmp" TMP="$WORKSPACE/tmp" TEMP="$WORKSPACE/tmp"
    # Fresh volume: seed ~/.bashrc and ~/.profile from the image defaults
    [ -f "$HOME/.bashrc" ] || cp /etc/skel/.bashrc "$HOME/.bashrc"
    [ -f "$HOME/.profile" ] || cp /etc/skel/.profile "$HOME/.profile"
fi
case ":$PATH:" in *":$HOME/.local/bin:"*) ;; *) export PATH="$HOME/.local/bin:$PATH" ;; esac
EOF
# Drop the default HIST*SIZE lines (they would override the values above) and the
# "interactive shells only" guards, so every command sees the same environment.
RUN sed -i '/^HIST\(FILE\)\?SIZE=/d' /root/.bashrc /etc/skel/.bashrc && \
    sed -i '/^\[ -z "\$PS1" \] && return/d' /etc/bash.bashrc /root/.bashrc /etc/skel/.bashrc && \
    sed -i '/^case \$- in/,/^esac/d' /root/.bashrc /etc/skel/.bashrc
# Every `docker run` loads the same bashrc files an interactive shell does, then runs the command.
# A bare interactive `bash` (the default CMD) reads them itself, so they are not loaded twice.
RUN cat > /usr/local/bin/entrypoint.sh <<'EOF'
#!/bin/bash
if ! { [ "$#" -eq 1 ] && { [ "$1" = /bin/bash ] || [ "$1" = bash ]; }; }; then
    [ -f /etc/bash.bashrc ] && . /etc/bash.bashrc
    [ -f "$HOME/.bashrc" ] && . "$HOME/.bashrc"
fi
exec "$@"
EOF
RUN chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["/bin/bash"]
