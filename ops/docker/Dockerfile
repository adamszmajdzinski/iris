# Base image
FROM ubuntu:20.04

# Install tzdata
RUN set dpkg to non-interactive for tzdata
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata

# Install other required software
RUN apt-get install -y \
    curl \
    sudo \
    gnupg2 \
    uwsgi \
    unzip \
    sudo \
    python3-pip \
    virtualenv \
    python3-dev \
    libyaml-dev \
    libsasl2-dev \
    libldap2-dev \
    uwsgi-plugin-python3 \
    uwsgi-plugin-gevent-python3 \
    mysql-client \
    git \
    vim \
    nginx

# Install Python3.7
RUN curl -fsSL https://keyserver.ubuntu.com/pks/lookup\?op\=get\&search\=0xba6932366a755776 | apt-key add -
RUN echo 'deb http://ppa.launchpad.net/deadsnakes/ppa/ubuntu focal main' > /etc/apt/sources.list.d/deadsnakes.list
RUN apt-get update && apt-get install -y \
    python3.7-dev \
    python3.7-venv

# Get Iris
RUN cd /srv && git clone https://github.com/linkedin/iris

# Setup Iris user
RUN useradd -m -s /bin/bash iris
#RUN chown -R iris:iris /home/iris
RUN ln -s /srv/iris /tmp/repo
RUN mv /tmp/repo /home/iris/source

# Setup Iris service
RUN cp /srv/iris/ops/config/systemd/uwsgi-iris.service /etc/systemd/system/uwsgi-iris.service

# Prepare Iris working dir
USER iris
RUN mkdir -p /home/iris/var/log/uwsgi /home/iris/var/run /home/iris/daemons /home/iris/config 

RUN virtualenv --python python3.7 /home/iris/env
RUN ln -s /home/iris/source/src/iris /home/iris/env/lib/python3.7/site-packages/iris
RUN ln -s /home/iris/source/ops/entrypoint.py /home/iris/entrypoint.py

WORKDIR /home/iris
USER root
RUN chown -R iris:iris /home/iris

# Setup Iris app inside virtualenv
RUN . /home/iris/env/bin/activate && cd /home/iris/source && python3.7 setup.py install && pip3.7 install uwsgi

# Run Iris
CMD ["/bin/bash", "-c", ". env/bin/activate && env/bin/uwsgi --yaml /home/iris/daemons/uwsgi.yaml:prod"]