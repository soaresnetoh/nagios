FROM ubuntu:22.04 as builder-base 

ARG TARGETPLATFORM
ARG BUILDPLATFORM

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

RUN addgroup ${NAGIOS_GROUP} && \
    useradd ${NAGIOS_USER} -g ${NAGIOS_CMDGROUP} -G ${APACHE_GROUP}

RUN apt update && \
    apt upgrade -y && \ 
    apt install -y software-properties-common \
    ca-certificates lsb-release apt-transport-https && \
    LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php && \
    apt update
    # mariadb-plugin-connect
RUN apt install -y git curl unzip apache2 apache2-utils rsyslog \
                        php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc \
                        php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev \
                        php7.4-imap php7.4-mbstring php7.4-opcache php7.4-soap php7.4-zip \
                        php7.4-intl php7.4 \ 
                        runit parallel ssmtp \
                        libltdl-dev libintl-perl libssl-dev procps tzdata \
                        libldap-dev mariadb-plugin-connect libfreeradius-dev libpq-dev libdbi-dev \
                        lm-sensors perl libnet-snmp-perl libsnmp-perl libcrypt-x509-perl \
                        libdatetime-perl libwww-perl libtext-glob-perl samba-client openssh-server openssl \
                        snmp bind9 bind9utils bind9-doc dnsutils && \
                                                \
    : '# For x64 the binary is : gosu-amd64' && \
    : '# For arm-v6 the binary is : gosu-armel' && \
    : '# For arm-v7 the binary is : gosu-armhf' && \
    : '# For arm64 the binary is : gosu-arm64' && \
    : '#######################################' && \
    : '# Creating an associative array with the platforms and their respective gosu release DOES NOT WORK in /bin/sh' && \
    echo "Arguments TARGETPLATFORM: ${TARGETPLATFORM} and BUILDPLATFORM: ${BUILDPLATFORM}" && \
    echo "$TARGETPLATFORM" | awk '{ gosuBinArr["linux/amd64"]="gosu-amd64"; gosuBinArr["linux/arm/v6"]="gosu-armel"; gosuBinArr["linux/arm/v7"]="gosu-armhf"; gosuBinArr["linux/arm64"]="gosu-arm64"; print gosuBinArr[$0];}' > mygosuver.txt && \
    gosuPlatform=$(cat mygosuver.txt) && \
    echo "Downloading ${gosuPlatform} for platform $TARGETPLATFORM" &&\
    curl -L -o gosu "https://github.com/tianon/gosu/releases/download/1.13/${gosuPlatform}"  && \
    mv gosu /bin/ && \
    chmod 755 /bin/gosu && \
    chmod +s /bin/gosu

RUN addgroup ${NAGIOS_GROUP} || true && \
    useradd ${NAGIOS_USER} -g ${NAGIOS_CMDGROUP} -G ${APACHE_GROUP}  || true 
    


# RUN apt update && \
#     apt install -y \
#     apache2 \
#     libapache2-mod-php \
#     php \
#     wget \
#     unzip \
#     zip \
#     autoconf \
#     gcc \
#     libc6 \
#     make \
#     apache2-utils \
#     libgd-dev

### ================================== ###
###   STAGE 2 COMPILE NAGIOS SOURCES   ###
### ================================== ###

FROM builder-base as builder-compile
ARG TARGETPLATFORM
ARG BUILDPLATFORM

# Add dependencies required to build Nagios
RUN apt update && \
    apt install -y  apache2 build-essential automake libtool autoconf python3-docutils gnutls-bin \
                        gnutls-dev g++ make gcc autoconf \
                        gettext linux-headers-generic openssl snmp \
                        libcrypto++-utils libpq-dev musl-dev libssl-dev libdbi-dev
RUN apt install -y libfreeradius-dev mariadb-plugin-connect \
                        openssh-client bind9 bind9utils bind9-doc dnsutils samba-client fping grep rpcbind \
                        lm-sensors snmp \
                        file perl \
                        libldap-dev postgresql-common

# RUN apt install -y git curl unzip apache2 apache2-utils rsyslog \
#                         php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc \
#                         php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev \
#                         php7.4-imap php7.4-mbstring php7.4-opcache php7.4-soap php7.4-zip \
#                         php7.4-intl php7.4 \ 
#                         runit parallel ssmtp \
#                         libltdl-dev libintl-perl libssl-dev procps tzdata \
#                         libldap-dev mariadb-plugin-connect libfreeradius-dev libpq-dev libdbi-dev \
#                         lm-sensors perl libnet-snmp-perl libsnmp-perl libcrypt-x509-perl \
#                         libdatetime-perl libwww-perl libtext-glob-perl samba-client openssh-server openssl \
#                         snmp bind9 bind9utils bind9-doc dnsutils && \




# Download Nagios core, plugins and nrpe sources                        
RUN    cd /tmp && \
       echo -n "Downloading Nagios ${NAGIOS_VERSION} source code: " && \
       wget -O nagios-core.tar.gz "https://github.com/NagiosEnterprises/nagioscore/archive/nagios-${NAGIOS_VERSION}.tar.gz" && \
       echo -n -e "OK\nDownloading Nagios plugins ${NAGIOS_PLUGINS_VERSION} source code: " && \
       wget -O nagios-plugins.tar.gz "https://github.com/nagios-plugins/nagios-plugins/archive/release-${NAGIOS_PLUGINS_VERSION}.tar.gz" && \
       echo -n -e "OK\nDownloading NRPE ${NRPE_VERSION} source code: " && \
       wget -O nrpe.tar.gz "https://github.com/NagiosEnterprises/nrpe/archive/nrpe-${NRPE_VERSION}.tar.gz" && \
       env && \
       echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM !! == " && \
       echo "OK"

# Compile Nagios Core
RUN    ls -l /tmp && cd /tmp && \
       tar zxf nagios-core.tar.gz && \
       tar zxf nagios-plugins.tar.gz && \
       tar zxf nrpe.tar.gz && \
       cd  "/tmp/nagioscore-nagios-${NAGIOS_VERSION}" && \
       echo -e "\n ===========================\n  Configure Nagios Core\n ===========================\n" && \
       ./configure \
            --prefix=${NAGIOS_HOME}                  \
            --exec-prefix=${NAGIOS_HOME}             \
            --enable-event-broker                    \
            --with-command-user=${NAGIOS_CMDUSER}    \
            --with-command-group=${NAGIOS_CMDGROUP}  \
            --with-nagios-user=${NAGIOS_USER}        \
            --with-nagios-group=${NAGIOS_GROUP}      && \
       : 'Apply patches to Nagios Core sources:' && \
       echo -n "Replacing \"<sys\/poll.h>\" with \"<poll.h>\": " && \
       sed -i 's/<sys\/poll.h>/<poll.h>/g' ./include/config.h && \
       echo "OK" && \
       echo -e "\n\n ===========================\n Compile Nagios Core\n ===========================\n" && \
       make all && \
       echo -e "\n\n ===========================\n  Install Nagios Core\n ===========================\n" && \
       make install && \
       make install-commandmode && \
       make install-config && \
       make install-webconf && \
       echo -n "Nagios installed size: " && \
       du -h -s ${NAGIOS_HOME}

# Compile Nagios Plugins
RUN    echo -e "\n\n ===========================\n  Configure Nagios Plugins\n ===========================\n" && \
       cd  /tmp/nagios-plugins-release-${NAGIOS_PLUGINS_VERSION} && \
       ./autogen.sh && \
       ./configure  --with-nagios-user=${NAGIOS_USER} \
                    --with-nagios-group=${NAGIOS_USER} \
                    --with-openssl \
                    --prefix=${NAGIOS_HOME}                                 \
                    --with-ping-command="/bin/gosu root /bin/ping -n -w %d -c %d %s"  \
                    --with-ipv6                                             \
                    --with-ping6-command="/bin/gosu root /bin/ping6 -n -w %d -c %d %s"  && \
       echo "Nagios plugins configured: OK" && \
       echo -n "Replacing \"<sys\/poll.h>\" with \"<poll.h>\": " && \
       egrep -rl "\<sys\/poll.h\>" . | xargs sed -i 's/<sys\/poll.h>/<poll.h>/g' && \
       egrep -rl "\"sys\/poll.h\"" . | xargs sed -i 's/"sys\/poll.h"/"poll.h"/g' && \
       echo "OK" && \
       echo -e "\n\n ===========================\n Compile Nagios Plugins\n ===========================\n" && \
       make && \
       echo "Nagios plugins compile successfully: OK" && \
       echo -e "\n\n ===========================\nInstall Nagios Plugins\n ===========================\n" && \
       make install && \
       echo "Nagios plugins installed successfully: OK"

# Compile NRPE
RUN    echo -e "\n\n =====================\n  Configure NRPE\n =====================\n" && \
       cd  /tmp/nrpe-nrpe-${NRPE_VERSION} && \
       ./configure --enable-command-args \
                    --with-nagios-user=${NAGIOS_USER} \
                    --with-nagios-group=${NAGIOS_USER} \
                    --with-ssl=/usr/bin/openssl \
                    --with-ssl-lib=/usr/lib
RUN  echo "NRPE client configured: OK" && \
       echo -e "\n\n ===========================\n  Compile NRPE\n ===========================\n" && \
       find / -iname "*check_nrpe*"
# RUN    cd /tmp/nrpe-nrpe-4.0.3/src/ && make check_nrpe
WORKDIR /tmp/nrpe-nrpe-4.0.3/
RUN    cd src && make check_nrpe
RUN    echo "NRPE compiled successfully: OK" && \
       echo -e "\n\n ===========================\n  Install NRPE\n ===========================\n"
       # make install && \
       
RUN    cp src/check_nrpe ${NAGIOS_HOME}/libexec/ && \
       echo "NRPE installed successfully: OK" && \
       echo -n "Final Nagios installed size: " && \
       du -h -s ${NAGIOS_HOME} 

# Compile Nagios files
# Create SSMTP configuration

RUN ls /etc/apache2
# RUN sed -i.bak 's/.*\=www\-data//g' /etc/apache2/envvars
RUN export DOC_ROOT="DocumentRoot $(echo $NAGIOS_HOME/share)"                                        && \
    sed -i "s,DocumentRoot.*,$DOC_ROOT," /etc/apache2/apache2.conf                                     && \
    sed -i "s|^ *ScriptAlias.*$|ScriptAlias /cgi-bin $NAGIOS_HOME/sbin|g" /etc/apache2/apache2.conf    && \
    sed -i 's/^\(.*\)#\(LoadModule cgi_module\)\(.*\)/\1\2\3/' /etc/apache2/apache2.conf               && \
    echo "ServerName ${NAGIOS_FQDN}" >> /etc/apache2/apache2.conf

RUN sed -i 's,/bin/mail,/usr/bin/mail,' ${NAGIOS_HOME}/etc/objects/commands.cfg  && \
    sed -i 's,/usr/usr,/usr,'           ${NAGIOS_HOME}/etc/objects/commands.cfg  && \
                                                                                    \
    : '# Modify Nagios mail commands in order to work with SSMTP'         && \
    sed -i 's/^.*command_line.*Host Alert.*$//g' /opt/nagios/etc/objects/commands.cfg && \
    sed -i 's/^.*command_line.*Service Alert.*$//g' /opt/nagios/etc/objects/commands.cfg && \
    sed -i '/notify-host-by-email/a command_line /usr/bin/printf "%b" "Subject: $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$\\n\\n***** Nagios *****\\n\\nNotification Type: $NOTIFICATIONTYPE$\\nHost: $HOSTNAME$\\nState: $HOSTSTATE$\\nAddress: $HOSTADDRESS$\\nInfo: $HOSTOUTPUT$\\n\\nDate/Time: $LONGDATETIME$\\n" | /usr/sbin/sendmail -v $CONTACTEMAIL$' ${NAGIOS_HOME}/etc/objects/commands.cfg  && \
    sed -i '/notify-service-by-email/a command_line /usr/bin/printf "%b" "Subject: $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$\\n\\n***** Nagios *****\\n\\nNotification Type: $NOTIFICATIONTYPE$\\n\\nService: $SERVICEDESC$\\nHost: $HOSTALIAS$\\nAddress: $HOSTADDRESS$\\nState: $SERVICESTATE$\\n\\nDate/Time: $LONGDATETIME$\\n\\nAdditional Info:\\n\\n$SERVICEOUTPUT$\\n" | /usr/sbin/sendmail -v $CONTACTEMAIL$' ${NAGIOS_HOME}/etc/objects/commands.cfg

RUN echo "use_timezone=${NAGIOS_TIMEZONE}" >> ${NAGIOS_HOME}/etc/nagios.cfg && \
    sed -i 's/date_format=us/date_format=iso8601/g' ${NAGIOS_HOME}/etc/nagios.cfg

# Copy original configuration to /orig directory
RUN mkdir -p /orig/apache2                     && \
    cp -r /etc/apache2/*  /orig/apache2        && \
    cp -r ${NAGIOS_HOME}/etc  /orig/etc        && \
    cp -r ${NAGIOS_HOME}/var  /orig/var         

### ========================== ###
### START OF ACTUAL DOCKERFILE ###
### ========================== ###

FROM builder-base

LABEL name="Nagios" \
      nagiosVersion=$NAGIOS_VERSION \
      nagiosPluginsVersion=$NAGIOS_PLUGINS_VERSION \
      nrpeVersion=$NRPE_VERSION \
      homepage="https://www.nagios.com/" \
      maintainer="Christos Manios <maniopaido@gmail.com>" \
      build="9"

RUN mkdir -p ${NAGIOS_HOME}  && \
    mkdir -p /orig/apache2

WORKDIR ${NAGIOS_HOME}
COPY --from=builder-compile ${NAGIOS_HOME} ${NAGIOS_HOME}

COPY --from=builder-compile /orig /orig

ADD overlay/ /

# Make 
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
            # dos2unix /etc/rsyslog.conf                && \
            # dos2unix /usr/local/bin/start_nagios      && \
            # dos2unix /etc/sv/**/run                   && \
            # dos2unix /etc/ssmtp/ssmtp.conf            && \
            : '# Add mail symbolic links to ssmtp'   && \
            ln -s $(which ssmtp) /bin/mail            && \
            ln -s $(which ssmtp) /usr/sbin/mail
            
            
EXPOSE 80

VOLUME "${NAGIOS_HOME}/var" "${NAGIOS_HOME}/etc" "/var/log/apache2" "/opt/Custom-Nagios-Plugins"

CMD [ "/usr/local/bin/start_nagios" ]
