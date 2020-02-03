    DOCKER DEAMON ACTIVATION PROCESS :


  ### These commands get run inside of your VM ###

#### Create the directory to store the configuration file ####
    sudo mkdir -p /etc/systemd/system/docker.service.d

#### Create a new file to store the daemon options  ####
    sudo vi /etc/systemd/system/docker.service.d/options.conf

#### Now make it look like this and save the file when you're done ####
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H unix:// -H tcp://0.0.0.0:2375   
#### Above command stats that , we are in open mode here we need to build security by generating the certificates ####
####    and need to set tha path   #### 

#### Reload the systemd daemon ####
    sudo systemctl daemon-reload

#### Restart Docker  ####
    sudo systemctl restart docker
#### -------------------------------------------------------------------------------------------------------------------- ####

    CERTIFICATES FOR DEAMON SECUITY :


    openssl genrsa -aes256 -out ca-key.pem 4096

    openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

    openssl genrsa -out server-key.pem 4096

    openssl req -subj "/CN=10.1.1.64" -sha256 -new -key server-key.pem -out server.csr

    echo subjectAltName = DNS:10.1.1.64,IP:10.1.1.64,IP:127.0.0.1 >> extfile.cnf

    echo extendedKeyUsage = serverAuth >> extfile.cnf

    openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
      -CAcreateserial -out server-cert.pem -extfile extfile.cnf
   
   
     openssl genrsa -out key.pem 4096 

     openssl req -subj '/CN=client' -new -key key.pem -out client.csr

    echo extendedKeyUsage = clientAuth > extfile-client.cnf 
    
     openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
      -CAcreateserial -out cert.pem -extfile extfile-client.cnf

    rm -v client.csr server.csr extfile.cnf extfile-client.cnf

    chmod -v 0400 ca-key.pem key.pem server-key.pem

    chmod -v 0444 ca.pem server-cert.pem cert.pem
    ----------------------------------------------------------------------------------------------------------
    # Here we will get almost 6 certificates , below are mapping convention for jenkins configuration we need to follow.

    Client Key : key.pem

    Client certificate: cert.pem

    Server CA certificate : server-cert.pem
    ----------------------------------------------------------------------------------------------------------
    # after certification generation , we need to map this cert in path as below command:

    and this [service]-tag commands we need to place in this path sudo vi /etc/systemd/system/docker.service.d/options.conf

    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/root/ca.pem --tlscert=/root/server-cert.pem --tlskey=/root/server-key.pem \
      -H=0.0.0.0:2376
    -----------------------------------------------------------------------------------------------------------
    #To Work docker commands we need to use below command to interact with deamon

    docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem \
      -H=$HOST:2376 version
    docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem \
      -H=10.1.3.122:2376 ps


     ------------------------------------ Please execute below process for docker ps and docker images --------------------------


    If you want to secure your Docker client connections by default, you can move the files to the .docker directory in your home directory --- and set the DOCKER_HOST and DOCKER_TLS_VERIFY variables as well (instead of passing -H=tcp://$HOST:2376 and --tlsverify on every call).

    $ mkdir -pv ~/.docker
    $ cp -v {ca,cert,key}.pem ~/.docker

    $ export DOCKER_HOST=tcp://$HOST:2376 DOCKER_TLS_VERIFY=1
    Docker now connects securely by default:

    $ docker ps
