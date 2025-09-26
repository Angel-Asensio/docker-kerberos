# docker-kerberos

This project is an example MIT Kerberos v5 containerization.

It is intended for demonstration / learning on a local host and is not production ready. In particular weak passwords are used.

It has the following components:

* compose file: https://github.com/Angel-Asensio/docker-kerberos/blob/master/docker-compose.yml
* Kerberos Administration Server: https://hub.docker.com/r/mans0954/kerberos-kadmin/
* Kerberos v5 Authentication Service and Key Distribution Center https://hub.docker.com/r/mans0954/kerberos-kdc/

## Run (MACOS) in detached mode (-d)
```
docker compose up -d 
```

## Check docker container(s) status
```
docker ps
```
=>
```
CONTAINER ID   IMAGE                      COMMAND                  CREATED       STATUS       PORTS                                    NAMES
33ee39caf7e8   mans0954/kerberos-kadmin   "/bin/sh -c '/usr/sb…"   3 hours ago   Up 3 hours   0.0.0.0:749->749/tcp                     kadmin
fa6f62b3f29c   mans0954/kerberos-kdc      "/bin/sh -c '/usr/sb…"   3 hours ago   Up 3 hours   0.0.0.0:88->88/tcp, 0.0.0.0:88->88/udp   kdc
```

## Administration
- Add a user principal using the admin interface and, optionally, an admin principal
```
alias kadmin.example.org="docker exec -ti kadmin kadmin.local"
# Add a user principal 'angel'. Enter a password for angel@EXAMPLE.ORG
kadmin.example.org -q "add_principal angel@EXAMPLE.ORG"
# (Optional) add an admin principal
kadmin.example.org -q "add_principal angel/admin@EXAMPLE.ORG"
```
*N.B.* 
kadm5.acl is configured so that all principals ending /admin have admin rights

## Authenticate from host
* Install Kerberos client tools if not already present. 
For macos: 
```
brew install krb5
```

* Configure /etc/krb5.conf on your host to point to the local KDC. Create or edit with:
```
[libdefaults]
    default_realm = EXAMPLE.ORG
    dns_lookup_realm = false
    dns_lookup_kdc = false
    forwardable = true

[realms]
    EXAMPLE.ORG = {
        kdc = 127.0.0.1:88
        admin_server = 127.0.0.1:749
    }
```
Use 127.0.0.1 or localhost to avoid DNS resolution issues. If your Docker setup uses a specific IP (e.g., for bridged networking), use that instead.

* **Obtain a Ticket-Granting Ticket (TGT)**: Run *kinit* to authenticate as the user principal *angel* 
```
kinit angel@EXAMPLE.ORG
```
It should prompt for the password you set earlier.

If succesful, this obtains a TGT from the KDC.

* **Verify the ticket**: Check your ticket cache
```
klist
```
=>
```
Credentials cache: API:195A7016-062F-4153-89E7-01507F1E3D8C
        Principal: angel@EXAMPLE.ORG

  Issued                Expires               Principal
Sep 26 13:55:46 2025  Sep 26 23:55:42 2025  krbtgt/EXAMPLE.ORG@EXAMPLE.ORG
```

## Troubleshoot Common Issues

- **"Cannot contact any KDC"**: Ports not exposed or firewall blocking (check sudo ufw allow 88/udp, etc., or disable firewall temporarily).
- **"Pre-authentication failed"**: Wrong password or principal not added correctly.
- **"No such file or directory"**: krb5.conf missing or misconfigured.
- **"Clock skew too great"**: Fix time sync as above.