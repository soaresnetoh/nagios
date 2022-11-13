FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=America/Sao_Paulo \
    NAGIOS_HOME=/opt/nagios \
    NAGIOS_USER=nagios \
    NAGIOS_GROUP=nagios \
    NAGIOS_CMDUSER=nagios \
    NAGIOS_CMDGROUP=nagios \
    NAGIOS_TIMEZONE=America/Sao_Paulo \
    NAGIOS_FQDN=nagios.example.com \
    NAGIOSADMIN_USER=nagiosadmin \
    NAGIOSADMIN_PASS=nagios \
    NAGIOS_VERSION=4.4.7 \
    NAGIOS_PLUGINS_VERSION=2.4.0 \
    NRPE_VERSION=4.0.3 \
    APACHE_LOCK_DIR=/var/run \
    APACHE_LOG_DIR=/var/log/apache2 \
    APACHE_GROUP=www-data


RUN apt update && \
    apt install -y \
    apache2 \
    libapache2-mod-php \
    php \
    wget \
    unzip \
    zip \
    autoconf \
    gcc \
    libc6 \
    make \
    apache2-utils \
    libgd-dev

RUN addgroup ${NAGIOS_GROUP} && \
    useradd ${NAGIOS_USER} -g ${NAGIOS_CMDGROUP} -G ${APACHE_GROUP}

ADD https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz /tmp
WORKDIR /tmp
RUN tar xzvf nagios-4.4.6.tar.gz && \
    tar xzvf nagios-4.4.6.tar.gz &&
    cd nagios-4.4.6 && \
    ./configure --with-httpd-conf=/etc/apache2/sites-enabled && \
    make all && \
    make install && \
    make install-init && \
    make install-commandmode && \
    make install-config && \
    make install-webconf && \
    htpasswd -c -b /usr/local/nagios/etc/htpasswd.users nagiosadmin nagios && \
    a2enmod cgi
ADD overlay/ /

RUN chmod +x /usr/local/bin/start_nagios                 \
            /etc/sv/apache/run                           \
            /etc/sv/nagios/run                           \
            /etc/sv/rsyslog/run                       && \
            rm -rf /etc/sv/getty-5                    && \
                                                         \
            : '# enable all runit services'           && \
            ln -s /etc/sv/* /etc/service              && \
                                                         \
            : '# Copy initial settings files'         && \
            chown -R nagios:nagios ${NAGIOS_HOME}     && \
            : '# Create special dirs'                 && \
            (mkdir /run/apache2 || true)              && \
            mkdir -p /var/spool/rsyslog               && \
            : '# Copy Apache configuration'           && \
            cp -Rp /orig/apache2/* /etc/apache2       && \
            : '# Convert files to Unix format'        && \
            dos2unix /etc/rsyslog.conf                && \
            dos2unix /usr/local/bin/start_nagios      && \
            dos2unix /etc/sv/**/run                   && \
            dos2unix /etc/ssmtp/ssmtp.conf            && \
            : '# Add mail symbolic links to ssmtp'   && \
            ln -s $(which ssmtp) /bin/mail            && \
            ln -s $(which ssmtp) /usr/sbin/mail



CMD [ "/usr/local/bin/start_nagios" ]



    systemctl enable nagios.service