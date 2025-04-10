# --------------------------------------------------------------------------------
# Docker configuration for P4D
# --------------------------------------------------------------------------------

FROM ubuntu:noble

LABEL vendor="Sourcegraph"
LABEL maintainer="Joe Chen (joe@sourcegraph.com)"

# Update Ubuntu and add Perforce Package Source
RUN apt-get update && \
  apt-get install -y curl gnupg2 && \
  curl -sS https://package.perforce.com/perforce.pubkey | gpg --dearmor | tee /etc/apt/trusted.gpg.d/perforce.gpg > /dev/null && \
  echo "deb http://package.perforce.com/apt/ubuntu noble release" > /etc/apt/sources.list.d/perforce.list && \
  apt-get update

# --------------------------------------------------------------------------------
# Docker BUILD
# --------------------------------------------------------------------------------

# Create perforce user and install Perforce Server
# Do in-page search over https://package.perforce.com/apt/ubuntu/dists/focal/release/binary-amd64/Packages
# for both "Package: helix-p4d" and "Package: helix-swarm-triggers".
RUN apt-get install -y helix-p4d=2024.2-2726408~noble helix-swarm-triggers=2024.6-2710109~noble

# Add external files
COPY files/restore.sh /usr/local/bin/restore.sh
COPY files/setup.sh /usr/local/bin/setup.sh
COPY files/init.sh /usr/local/bin/init.sh
COPY files/latest_checkpoint.sh /usr/local/bin/latest_checkpoint.sh

RUN \
  chmod +x /usr/local/bin/restore.sh && \
  chmod +x /usr/local/bin/setup.sh && \
  chmod +x /usr/local/bin/init.sh && \
  chmod +x /usr/local/bin/latest_checkpoint.sh

# --------------------------------------------------------------------------------
# Docker ENVIRONMENT
# --------------------------------------------------------------------------------

# Default Environment
ARG NAME=helix
ARG P4NAME=main
ARG P4TCP=1666
ARG P4USER=admin
ARG P4PASSWD=temporary99
ARG P4CASE=-C0
ARG P4CHARSET=utf8

# Dynamic Environment
ENV NAME=$NAME \
  P4NAME=$P4NAME \
  P4TCP=$P4TCP \
  P4PORT=$P4TCP \
  P4USER=$P4USER \
  P4PASSWD=$P4PASSWD \
  P4CASE=$P4CASE \
  P4CHARSET=$P4CHARSET \
  JNL_PREFIX=$P4NAME

# Base Environment
ENV P4HOME=/p4

# Derived Environment
ENV P4ROOT=$P4HOME/root \
  P4DEPOTS=$P4HOME/depots \
  P4CKP=$P4HOME/checkpoints

# Expose Perforce; TCP port and volumes
EXPOSE $P4TCP
VOLUME $P4HOME

# --------------------------------------------------------------------------------
# Docker RUN
# --------------------------------------------------------------------------------

ENTRYPOINT \
  init.sh && \
  /usr/bin/tail -F $P4ROOT/logs/log

HEALTHCHECK \
  --interval=2m \
  --timeout=10s \
  CMD p4 info -s > /dev/null || exit 1
