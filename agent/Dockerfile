FROM python:3.9.7-buster AS builder

# docker-compose requires pyinstaller 3.6 (check github.com/docker/compose/requirements-build.txt)
ARG VERSION=1.29.2
ARG PYINSTALLER_VER=4.1

ENV LANG C.UTF-8

RUN mkdir -p /build/pyinstallerbootloader && \
		# Compile the pyinstaller "bootloader"
		# https://pyinstaller.readthedocs.io/en/stable/bootloader-building.html
		cd /build/pyinstallerbootloader && \
		curl -sSL https://github.com/pyinstaller/pyinstaller/releases/download/v$PYINSTALLER_VER/PyInstaller-$PYINSTALLER_VER.tar.gz | tar xz --strip 1 && \
		cd bootloader && python3 ./waf all && \
		# Clone docker-compose
		mkdir -p /build/dockercompose && \
		cd /build/dockercompose && \
		git clone https://github.com/docker/compose.git . && \
		git checkout $VERSION && \
		# ./script/build/write-git-sha && \
		git rev-parse --short HEAD > compose/GITSHA && \
		# Run the build steps (taken from github.com/docker/compose/script/build/linux-entrypoint)
		mkdir ./dist && \
		pip install -q -r requirements.txt -r requirements-build.txt && \
		pyinstaller docker-compose.spec && \
		mv dist/docker-compose /usr/bin/docker-compose

# https://github.com/jenkinsci/docker-agent/blob/master/11/alpine/Dockerfile
# https://github.com/jenkinsci/docker-inbound-agent/blob/master/11/alpine/Dockerfile
FROM adoptopenjdk/openjdk11:alpine-slim

COPY --from=builder /usr/bin/docker-compose /usr/bin/docker-compose

ARG VERSION=4.10
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

RUN addgroup -g ${gid} ${group}
RUN adduser -h /home/${user} -u ${uid} -G ${group} -D ${user}
LABEL Description="This is a base image, which provides the Jenkins agent executable (slave.jar)" Vendor="Jenkins project" Version="${VERSION}"

ARG AGENT_WORKDIR=/home/${user}/agent
COPY jenkins-agent /usr/local/bin/jenkins-agent

RUN apk add --update --no-cache \
		curl bash git git-lfs openssh-client openssl procps \
		\
		maven docker py3-pip go \
	\
		&& pip install awscli \
	\
		# Install kubectl
		&& curl -Lo /usr/bin/kubectl "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" \
		&& chmod +x /usr/bin/kubectl \
  \
		&& curl --create-dirs -fsSLo /usr/share/jenkins/agent.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${VERSION}/remoting-${VERSION}.jar \
		&& chmod 755 /usr/share/jenkins \
		&& chmod 644 /usr/share/jenkins/agent.jar \
		&& ln -sf /usr/share/jenkins/agent.jar /usr/share/jenkins/slave.jar \
		&& apk del curl \
		\
		&& chmod +x /usr/local/bin/jenkins-agent \
		&& ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave
#USER ${user}
USER root
ENV AGENT_WORKDIR=${AGENT_WORKDIR}
RUN mkdir /home/${user}/.jenkins && mkdir -p ${AGENT_WORKDIR}

VOLUME /home/${user}/.jenkins
VOLUME ${AGENT_WORKDIR}
WORKDIR /home/${user}

ENTRYPOINT ["jenkins-agent"]

