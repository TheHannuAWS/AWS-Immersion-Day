FROM ubuntu:focal
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -y net-tools iputils-ping iproute2 python python3 pip vim
RUN pip install boto3 requests
WORKDIR /
