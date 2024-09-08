# Install Artifactory on a Proxmox VM which runs AlmaLinux

## Install PostgreSQL database

To install PostgreSQL I used the DNF modularity command. 

```bash
dnf module install postgresql:15
```

I installed the PostgreSQL 15 version because this is the highest version which is supported by the Artifactory. After
the installation is completed, we have to initialize the database with the command below.

```bash
sudo /usr/bin/postgresql-setup --initdb
```

Finally I enabled the `postgresql` service in order to start automatically at system boot and then start the service.

```bash
sudo systemctl start postgresql
sudo systemctl status postgresql
```

After the service is up and running the firewall needs some tweak too.

```bash
sudo firewall-cmd --add-service=postgresql --permanent
sudo firewall-cmd --reload
```

## Install Artifactory

### Install Artifactory package

To install the `Artifactory` you don't have to do anything big, just visit the [Artifactory download page](https://jfro
g.com/community/download-artifactory-oss/) and choose the right installer for you. I chose the `RPM` installer from the 
`Linux instlalers` dropdown and clickthe copy command link. It copies the install command to the clipboard which are 
below.

My fresh AlmaLinux installation is lack of the `wget` command by default, so first I installed it.

```bash
sudo dnf install wget
```

```bash
wget https://releases.jfrog.io/artifactory/artifactory-rpms/artifactory-rpms.repo -O jfrog-artifactory-rpms.repo;
sudo mv jfrog-artifactory-rpms.repo /etc/yum.repos.d/;
sudo dnf update && sudo dnf install jfrog-artifactory-oss
```

### Prepare database

I had to prepare the database in order to the Artifactory will be able to connect to it. So I created a user and a 
database, then grant privileges to the created user on the created database.

```bash
CREATE USER artifactory WITH PASSWORD 'password';
CREATE DATABASE artifactory WITH OWNER=artifactory ENCODING='UTF8';
GRANT ALL PRIVILEGES ON DATABASE artifactory TO artifactory;
```

Then changed the authentication method from `ident` to `md5` in the `pg_hba.conf` file in order to PostgreSQL server
accepts the password authentication from the Artifactory.

### Configure Artifactory to use the PostgreSQL installation

I searched the file named `system.yaml` which stores the configuration of the Artifactory as well as the database
configurations. Here I specified the URL, the username and of course the password. After this I started and enabled
the Artifactory service.

```bash
sudo systemctl enable artifactory
sudo systemctl start artifactory
```

### Configure Artifactory where to store the files

After mounted the device where Artifactory should store the binaries find the `binarystore.xml` file, where specified
where the binaries and temporary files go.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<config version="1">
        <chain template="file-system"/>
        <provider id="file-system" type="file-system">
                <baseDataDir>/mnt/artficatory-store/</baseDataDir>
                <fileStoreDir>filestore</fileStoreDir>
                <tempDir>temp</tempDir>
        </provider>
</config>
```

## Install NGINX

```bash
dnf module install nginx:1.24
```

After the installation the NGINX proxy in AlmaLinux I had to enable the following SELinux flag in order to the NGINX
be able to make connections.

```bash
setsebool httpd_can_network_connect 1
```

Because I wanted to reach the Artifactory through HTTPS I had to generate a TLS keypair. I used the command below for
generate a self-signed keypair.

```bash
openssl req -newkey rsa:2048 -nodes -keyout artifactory.key -x509 -days 365 -out artifactory.crt
```

Last I used the following NGINX configuration which was placed under the `/etc/nginc/conf.d/` folder.

```plain
ssl_protocols        TLSv1.2 TLSv1.3;
ssl_certificate      /etc/pki/tls/certs/artifactory/artifactory.crt;
ssl_certificate_key  /etc/pki/tls/certs/artifactory/artifactory.key;
ssl_session_cache    shared:SSL:1m;
ssl_prefer_server_ciphers   on;

server {
    listen 80;
    server_name artifactory.kubenetic.home;

    return 301 https://$host$request_uri;
}

## server configuration
server {
    listen 443 ssl;

    server_name artifactory.kubenetic.home;

    if ($http_x_forwarded_proto = '') {
        set $http_x_forwarded_proto  $scheme;
    }

    ## Application specific logs
    access_log /var/log/nginx/artifactory-access.log;
    error_log /var/log/nginx/artifactory-error.log;

    rewrite ^/$ /ui/ redirect;
    rewrite ^/ui$ /ui/ redirect;

    chunked_transfer_encoding on;
    client_max_body_size 0;

    location / {
        proxy_read_timeout  2400s;
        proxy_pass_header   Server;
        proxy_cookie_path   ~*^/.* /;

        proxy_pass          http://127.0.0.1:8082;

        proxy_next_upstream error timeout non_idempotent;
        proxy_next_upstream_tries    1;

        proxy_set_header    X-JFrog-Override-Base-Url $http_x_forwarded_proto://$host:$server_port;
        proxy_set_header    X-Forwarded-Port  $server_port;
        proxy_set_header    X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_set_header    Host              $http_host;
        proxy_set_header    Authorization     $http_authorization;
        proxy_set_header    X-Forwarded-For   $proxy_add_x_forwarded_for;

        location ~ ^/artifactory/ {
            proxy_pass    http://127.0.0.1:8081;
        }
    }
}
```

## Verdict

Because of reasons the Artifactory worked only in Chromium well. In Firefox I got misterious HTTP401 and HTTP403 errors.
I was able to signin on the UI but if the UI wanted to send any API request to the server I was dropped out despite
I was logged in as an administrator. Maybe some cacheing or session handling issue.
