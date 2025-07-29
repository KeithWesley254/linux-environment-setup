# Jenkins Docker Agent Setup Guide

This document outlines the procedure for setting up a Jenkins agent using a custom Docker image. This approach provides a consistent and isolated environment for your Jenkins builds, pre-configured with necessary tools like various web browsers (Chrome, Firefox, Edge, Brave, Opera), Java, and Maven.

**Jenkins Version:** 2.516.1 (or similar)  
**Prerequisites:** Docker CLI and Docker Desktop installed on the host machine where the agent will run.

---

## 1. Prepare Your Custom Jenkins Agent Image

### 1.1 Dockerfile

```dockerfile
# Base Image
FROM ubuntu:22.04

LABEL maintainer="Keith Wesley <https://keithwesley254.github.io/>"

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV GECKODRIVER_VERSION=0.34.0

# Install essential packages and tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash curl wget unzip git gnupg ca-certificates \
    software-properties-common apt-transport-https \
    xvfb libxi6 libgconf-2-4 libnss3 libxss1 libasound2 \
    libgbm1 fonts-liberation libappindicator3-1 \
    chromium-driver openjdk-17-jdk maven sudo libvulkan1 \
    xdg-utils libatk-bridge2.0-0 libatk1.0-0 libcups2 \
    libdbus-1-3 libgdk-pixbuf2.0-0 libnspr4 libxcomposite1 \
    libxdamage1 libxrandr2 && rm -rf /var/lib/apt/lists/*

# Google Chrome
RUN wget -q -O chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    && apt-get update && apt-get install -y ./chrome.deb \
    && rm chrome.deb

# Firefox + Geckodriver
RUN apt-get update && apt-get install -y firefox \
    && curl -L "https://github.com/mozilla/geckodriver/releases/download/v$GECKODRIVER_VERSION/geckodriver-v$GECKODRIVER_VERSION-linux64.tar.gz" \
    | tar -xz -C /usr/local/bin \
    && chmod +x /usr/local/bin/geckodriver

# Microsoft Edge
RUN curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg \
    && install -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/ \
    && sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/edge stable main" > /etc/apt/sources.list.d/microsoft-edge.list' \
    && rm microsoft.gpg \
    && apt-get update && apt-get install -y microsoft-edge-stable

# Brave
RUN curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg] https://brave-browser-apt-release.s3.brave.com/ stable main" \
    | tee /etc/apt/sources.list.d/brave-browser-release.list \
    && apt-get update && apt-get install -y brave-browser

# Opera
RUN wget -qO- https://deb.opera.com/archive.key | gpg --dearmor | tee /usr/share/keyrings/opera.gpg > /dev/null \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/opera.gpg] https://deb.opera.com/opera-stable/ stable non-free" \
    > /etc/apt/sources.list.d/opera.list \
    && apt-get update && apt-get install -y opera-stable

# Jenkins user and agent setup
RUN useradd -m -d /home/jenkins -s /bin/bash jenkins \
    && mkdir -p /home/jenkins/agent \
    && chown -R jenkins:jenkins /home/jenkins

COPY agent.jar /home/jenkins/agent.jar
COPY entrypoint.sh /home/jenkins/entrypoint.sh
RUN chmod +x /home/jenkins/entrypoint.sh \
    && chown -R jenkins:jenkins /home/jenkins

USER jenkins
WORKDIR /home/jenkins

ENTRYPOINT ["./entrypoint.sh"]
```

### 1.2 `entrypoint.sh`

```bash
#!/bin/bash

: "${JENKINS_URL:?JENKINS_URL must be set}"
: "${JENKINS_SECRET:?JENKINS_SECRET must be set}"
: "${JENKINS_AGENT_NAME:?JENKINS_AGENT_NAME must be set}"

exec java -jar /home/jenkins/agent.jar \
  -url "$JENKINS_URL" \
  -secret "$JENKINS_SECRET" \
  -name "$JENKINS_AGENT_NAME" \
  -workDir /home/jenkins/agent \
  -webSocket
```

### 1.3 Obtain `agent.jar`

Download from:  
`http://your-jenkins-ip:8080/jnlpJars/agent.jar`

Place the file in the same directory as your Dockerfile and `entrypoint.sh`.

### 1.4 Build the Docker Image

```bash
docker build -t my-jenkins-agent:latest .
```

---

## 2. Configure the Jenkins Node (Controller Side)

1. Navigate to **Manage Jenkins > Nodes**.
2. Click **New Node**.
3. Enter a name (e.g., `multi-browser-mavenjava-agent`), select **Permanent Agent**, and click **Create**.
4. Fill in the details:
    - Remote root directory: `/home/jenkins/agent`
    - Labels: `multi-browser-mavenjava-agent`
    - Launch method: **Launch agent via Java Web Start**
5. Click **Save**.

---

## 3. Retrieve Agent Connection Details

Go to the node page after saving. Copy the following values:

- **Jenkins URL** (e.g., `http://192.168.1.100:8080/` - Find your IP address and make it a string URL (localhost))
- **Agent Name** (e.g., `multi-browser-mavenjava-agent`)
- **Secret** (a long token)

---

## 4. Run Your Docker Agent Container

Determine the host IP address:

### Example:
```bash
docker run -d \
  -e JENKINS_URL="http://192.168.1.100:8080/" \
  -e JENKINS_SECRET="xxxxxxxxxxxxxxxxxxxxx" \
  -e JENKINS_AGENT_NAME="multi-browser-mavenjava-agent" \
  -v /var/run/docker.sock:/var/run/docker.sock \  # optional
  keithwesley254/maven-docker-agent:latest #my docker hub image
```

---

## 5. Verify the Agent Connection

In Jenkins:
- Go to **Manage Jenkins > Nodes**
- Verify the agent shows as **online** with a green check (or at least doesnt have a red X)

You can now use the agent with:

```groovy
agent {
  label 'multi-browser-mavenjava-agent'
}
```

---

End of Guide.