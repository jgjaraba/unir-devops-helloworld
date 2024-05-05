FROM jenkins/ssh-agent

RUN apt-get update && apt-get install -y python3 python3-pip

RUN apt install -y python3-flask && apt install -y python3-pytest

EXPOSE 8080
EXPOSE 50000
EXPOSE 22
