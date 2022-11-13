no server - 
    docker run -d -it --name nagiosserver -h nagios -p 8181:80 appcontainers/nagios
    docker container exec -it nagiosserver htpasswd -c -b /etc/nagios/passwd nagiosadmin nagios
        mkdir -p /etc/nagios/servers
        cd /etc/nagios/servers
    criar arquivo 
        vim ubuntu_host.cfg
    restart serviço 
        service nagios restart



no host - 
    docker run -d -it --name nagioshost -p 0.0.0.0:2222:22 nuagebec/ubuntu 
    instalar 
        apt update && apt install -y nagios-nrpe-server nagios-plugins
    alterar o arquivo 
        cd /etc/nagios 
        vim nrpe.cfg #(server_address=172.17.0.2 / server_port=8181)
    restart serviço 
        service nagios-nrpe-server restart

