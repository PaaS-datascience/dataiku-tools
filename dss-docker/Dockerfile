FROM almalinux:9

ARG dssVersion

ENV DSS_VERSION="$dssVersion" \
    DSS_DATADIR="/home/dataiku/dss" \
    DSS_PORT=10000

# Dataiku account and data dir setup
RUN useradd dataiku \
    && mkdir -p /home/dataiku ${DSS_DATADIR} \
    && chown -Rh dataiku:dataiku /home/dataiku ${DSS_DATADIR}

# Download and extract DSS kit
RUN DSSKIT="dataiku-dss-$DSS_VERSION" \
    && cd /home/dataiku \
    && echo "+ Downloading kit" \
    && curl -OsS "https://cdn.downloads.dataiku.com/public/studio/$DSS_VERSION/$DSSKIT.tar.gz" \
    && echo "+ Extracting kit" \
    && tar xf "$DSSKIT.tar.gz" \
    && rm "$DSSKIT.tar.gz" \
    && "$DSSKIT"/scripts/install/install-deps.sh -yes -with-r -with-chrome \
    && "$DSSKIT"/scripts/install/installdir-postinstall.sh "$DSSKIT" \
    && chown -Rh dataiku:dataiku "$DSSKIT"

# Install required R packages
RUN mkdir -p /usr/local/lib/R/site-library \
    && R --slave --no-restore \
        -e "install.packages( \
            c('httr', 'RJSONIO', 'dplyr', 'curl', 'IRkernel', 'sparklyr', 'ggplot2', 'gtools', 'tidyr', \
            'tidyr', 'rmarkdown', 'base64enc', 'filelock'), \
            '/usr/local/lib/R/site-library', \
            repos='https://cloud.r-project.org')"

# Install Python 3.11
RUN PYTHON_VERSION=3.11.11 \
    && dnf install -y gcc openssl-devel bzip2-devel libffi-devel zlib-devel \
    && curl "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz" | \
    tar xzf - \
    && cd Python-${PYTHON_VERSION} \
    && ./configure --enable-optimizations \
    && make altinstall \
    && cd .. \
    && rm -rf Python-${PYTHON_VERSION}

# Enable docker to enable to build compatible docker images
# Don't need to install systemctl
RUN dnf --refresh update -y \
    && dnf upgrade -y \
    && dnf install -y yum-utils \
    && yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo \
    && dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin \
    && dnf clean all

# Enable kubectl management
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Entry point
WORKDIR /home/dataiku
USER dataiku

RUN DSSKIT="dataiku-dss-$DSS_VERSION" \
    && "$DSSKIT"/installer.sh -d "$DSS_DATADIR" -p "$DSS_PORT" \
    && "$DSS_DATADIR"/bin/dssadmin install-R-integration \
    && "$DSS_DATADIR"/bin/dssadmin install-graphics-export

COPY run.sh /home/dataiku/

EXPOSE $DSS_PORT

CMD [ "/home/dataiku/run.sh" ]
