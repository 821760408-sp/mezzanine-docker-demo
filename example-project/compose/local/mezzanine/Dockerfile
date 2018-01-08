FROM python:3.6-alpine


##################################################################################
# Dockerized SSH service, based on Dockerfile from:
# https://registry.hub.docker.com/u/rastasheep/ubuntu-sshd/dockerfile/
# Despite http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/
# I found, that this is an only way to connect to remote debugger
##################################################################################

RUN apk add --no-cache openssh-server
RUN mkdir /var/run/sshd

RUN echo 'root:root' |chpasswd

RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

EXPOSE 22

##################################################################################
# Mezzanine Server Stack
##################################################################################

RUN apk add --no-cache musl-dev
RUN apk add --no-cache nginx
RUN apk add --no-cache postgresql-dev
RUN apk add --no-cache memcached
RUN apk add --no-cache supervisor

##################################################################################
# Application staff
##################################################################################

RUN pip install Mezzanine==4.2.3
RUN pip install psycopg2==2.7.3.2
RUN pip install gunicorn==19.7.1
RUN pip install python-memcached==1.59
RUN pip install setproctitle==1.1.10

# Because of error: https://gist.github.com/noisy/d13c18f831bf1fc2c2d3
# See: https://github.com/gliderlabs/docker-alpine/issues/144

# tell Nginx to stay foregrounded
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

ADD ../../../ /mezzanine-project
WORKDIR /mezzanine-project
RUN pip install -r /mezzanine-project/requirements.txt

RUN mkdir -p /mezzanine-project/static/media


##################################################################################
# Application Production Configuration
##################################################################################

ADD ../../../deploy/docker/crontab /etc/cron.d/blog
ADD ../../../deploy/docker/gunicorn.conf.py /mezzanine-project/gunicorn.conf.py
ADD ../../../deploy/docker/local_settings.py /mezzanine-project/local_settings.py
ADD ../../../deploy/docker/nginx.conf /etc/nginx/sites-enabled/blog.conf
ADD ../../../deploy/docker/supervisor.conf /etc/supervisor/conf.d/blog.conf

CMD sleep 5 && \
    python manage.py collectstatic --noinput && \
    python manage.py syncdb --noinput && \
    python manage.py migrate --noinput && \
    /usr/bin/supervisord --nodaemon

ENV DEBIAN_FRONTEND newt
