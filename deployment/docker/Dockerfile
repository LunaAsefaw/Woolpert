FROM python:3.10.2-buster AS prod
LABEL GeoNode development team

RUN mkdir -p /usr/src/woolpert

# Enable postgresql-client-13
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ buster-pgdg main" | tee /etc/apt/sources.list.d/pgdg.list
RUN echo "deb http://deb.debian.org/debian/ stable main contrib non-free" | tee /etc/apt/sources.list.d/debian.list
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

# To get GDAL 3.2.1 to fix this issue https://github.com/OSGeo/gdal/issues/1692
# TODO: The following line should be removed if base image upgraded to Bullseye
RUN echo "deb http://deb.debian.org/debian/ bullseye main contrib non-free" | tee /etc/apt/sources.list.d/debian.list

# This section is borrowed from the official Django image but adds GDAL and others
RUN apt-get update -y && apt-get upgrade -y

# Prepraing dependencies
RUN apt-get install -y \
    libgdal-dev libpq-dev libxml2-dev \
    libxml2 libxslt1-dev zlib1g-dev libjpeg-dev \
    libmemcached-dev libldap2-dev libsasl2-dev libffi-dev

RUN apt-get install -y --no-install-recommends \
    gcc zip gettext geoip-bin cron \
    postgresql-client-13 \
    sqlite3 spatialite-bin libsqlite3-mod-spatialite \
    python3-dev python3-gdal python3-psycopg2 python3-ldap \
    python3-pip python3-pil python3-lxml python3-pylibmc \
    uwsgi uwsgi-plugin-python3 \
    firefox-esr

RUN apt-get install -y devscripts build-essential debhelper pkg-kde-tools sharutils
# RUN git clone https://salsa.debian.org/debian-gis-team/proj.git /tmp/proj
RUN apt-get install gdal-bin
# RUN cd /tmp/proj && debuild -i -us -uc -b && dpkg -i ../*.deb 

# Install pip packages
RUN pip install pip --upgrade \
    && pip install pygdal==$(gdal-config --version).* \
        flower==0.9.4

# Activate "memcached"
RUN apt install -y memcached
RUN pip install pylibmc \
    && pip install sherlock

# add bower and grunt command
COPY src /usr/src/woolpert/
WORKDIR /usr/src/woolpert

COPY src/monitoring-cron /etc/cron.d/monitoring-cron
RUN chmod 0644 /etc/cron.d/monitoring-cron
RUN crontab /etc/cron.d/monitoring-cron
RUN touch /var/log/cron.log
RUN service cron start

COPY src/wait-for-databases.sh /usr/bin/wait-for-databases
RUN chmod +x /usr/bin/wait-for-databases
RUN chmod +x /usr/src/woolpert/tasks.py \
    && chmod +x /usr/src/woolpert/entrypoint.sh

COPY src/celery.sh /usr/bin/celery-commands
RUN chmod +x /usr/bin/celery-commands

COPY src/celery-cmd /usr/bin/celery-cmd
RUN chmod +x /usr/bin/celery-cmd

# # Install "geonode-contribs" apps
# RUN cd /usr/src; git clone https://github.com/GeoNode/geonode-contribs.git -b master
# # Install logstash and centralized dashboard dependencies
# RUN cd /usr/src/geonode-contribs/geonode-logstash; pip install --upgrade  -e . \
#     cd /usr/src/geonode-contribs/ldap; pip install --upgrade  -e .
# RUN python -m venv venv
# RUN . ./venv/bin/activate
RUN pip install --upgrade --no-cache-dir  --src /usr/src -r requirements.txt
RUN pip install --upgrade  -e .
RUN python setup.py install

# Cleanup apt update lists
RUN rm -rf /var/lib/apt/lists/*

# Export ports
EXPOSE 8000

# We provide no command or entrypoint as this image can be used to serve the django project or run celery tasks
# ENTRYPOINT /usr/src/woolpert/entrypoint.sh

FROM prod AS dev

RUN pip install -r requirements-dev.txt
RUN pip install --upgrade  -e .

# This section taken on 2 July 2015 from
# https://docs.docker.com/examples/running_ssh_service/
# Sudo is needed by pycharm when it tries to pip install packages
RUN apt-get update && apt-get install -y openssh-server sudo
RUN mkdir -p /var/run/sshd
RUN echo 'root:docker' | chpasswd
RUN sed -i 's/^PermitRootLogin */#PermitRootLogin /' /etc/ssh/sshd_config
RUN echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# --------------------------------------------------------
# Open ports as needed
# --------------------------------------------------------

# Open port 8080 as we will be running our django dev server on
EXPOSE 8080
# Open port 22 as we will be using a remote interpreter from pycharm
EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]