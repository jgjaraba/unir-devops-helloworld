FROM jenkins/ssh-agent

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /opt/venv \
    && /opt/venv/bin/pip install --upgrade pip \
    && /opt/venv/bin/pip install flask \
    && /opt/venv/bin/pip install pytest \
    && /opt/venv/bin/pip install flake8 \
    && /opt/venv/bin/pip install bandit \
    && /opt/venv/bin/pip install coverage

EXPOSE 8080
EXPOSE 50000
EXPOSE 22
