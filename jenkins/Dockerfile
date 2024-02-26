# Verwenden des Jenkins LTS Docker Images als Basis
FROM jenkins/jenkins:lts

# Wechseln zum Root-Benutzer für die Installation
USER root

# Aktualisieren der Paketlisten und Installieren von Abhängigkeiten
RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-venv \
    curl \
    maven \
    git \
    perl \
    wget \
    kbtin \
    libnet-ssleay-perl \
    software-properties-common

# Hinzufügen des öffentlichen Schlüssels für das Multiverse-Repository, das Nikto enthält
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32 && \
    add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic multiverse' && \
    add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic-security multiverse' && \
    add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic-updates multiverse'

# Erneutes Aktualisieren der Paketlisten nach dem Hinzufügen der Repositories und Installieren von Nikto
RUN apt-get update -y && \
    apt-get install -y nikto

# Erstellen einer virtuellen Umgebung für Python-Tools
RUN python3 -m venv /opt/venv

# Aktivieren der virtuellen Umgebung und Installieren von Python-Paketen
RUN /bin/bash -c "source /opt/venv/bin/activate && \
    pip install --upgrade pip && \
    pip install virtualenv safety liccheck trufflehog bandit selenium ansible boto boto3"

# Kopieren der Ansible-Konfigurationsdatei in den Container
COPY ansible.cfg /etc/ansible/ansible.cfg

# Installieren der Jenkins-Plugins mit dem jenkins-plugin-cli Tool
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

# Zurückwechseln zum Jenkins-Benutzer
USER jenkins

