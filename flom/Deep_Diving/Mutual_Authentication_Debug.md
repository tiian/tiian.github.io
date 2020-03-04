# Mutual Authentication Debug
This debug session uses two different nodes; these are the names used during all the session:

* node1, name: *ubuntu1004.brenta.org*, operating system: Ubuntu Linux 10.04 (64 bit)
* node2, name: *centos71-64.brenta.org*, operating system: CentOS 7.1 (64 bit)

It is suggested to try [Channel Encryption Debug](Channel_Encryption_Debug.md) before proceeding with this more complex one.

## Native FLoM debugging tool

Sometimes debugging becomes difficult, especially when you have to deal with networks, firewalls, public and private addresses and so on.   
For some complex features, FLoM provide a native debugging tool that should help system engineers to understand the root cause of an issue.

## Setting up the certification authority and the required certificates

### Creating a CA (Certification Authority) of name *CA2*
Connect to node1, and execute these commands to create the directory structure:

~~~
tiian@ubuntu1004:~$ mkdir flom_ssl
tiian@ubuntu1004:~$ cd flom_ssl
tiian@ubuntu1004:~/flom_ssl$ cd CA2
tiian@ubuntu1004:~/flom_ssl/CA2$ ls
tiian@ubuntu1004:~/flom_ssl/CA2$ mkdir certs crl newcerts private
tiian@ubuntu1004:~/flom_ssl/CA2$ echo "01" > serial
tiian@ubuntu1004:~/flom_ssl/CA2$ cp /dev/null index.txt
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la
total 28
drwxr-xr-x 6 tiian tiian 4096 2016-03-30 21:54 .
drwxr-xr-x 4 tiian tiian 4096 2016-03-30 21:53 ..
drwxr-xr-x 2 tiian tiian 4096 2016-03-30 21:53 certs
drwxr-xr-x 2 tiian tiian 4096 2016-03-30 21:53 crl
-rw-r--r-- 1 tiian tiian    0 2016-03-30 21:54 index.txt
drwxr-xr-x 2 tiian tiian 4096 2016-03-30 21:53 newcerts
drwxr-xr-x 2 tiian tiian 4096 2016-03-30 21:53 private
-rw-r--r-- 1 tiian tiian    3 2016-03-30 21:54 serial
~~~

Pick-up a *openssl.cnf* example file; FLoM provide a pre-configured file in directory */usr/local/share/doc/flom/*:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la /usr/local/share/doc/flom/flom_openssl.conf
-rw-r--r-- 1 root root 9431 2016-03-28 19:34 /usr/local/share/doc/flom/flom_openssl.conf
~~~

copy it locally:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ cp /usr/local/share/doc/flom/flom_openssl.conf .
~~~

Generate the certificate for the CA:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -days 3650 -config flom_openssl.conf
Generating a 1024 bit RSA private key
...++++++
.................................................++++++
writing new private key to 'private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IT]:
State or Province Name (full name) [Treviso]:
Locality Name (eg, city) [Mogliano Veneto]:
Organization Name (eg, company) [FLoM Software Corporation]:
Organizational Unit Name (eg, section) [Development and Research]:
Common Name (eg, YOUR name) []:CA for FLoM Mutual Authentication
Email Address []:
~~~

File *cacert.pem* contains the **X.509 certificate** of the certification authority you have just created and file *private/cakey.pem* contains the **private key** associated to the certification authority.

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la cacert.pem private/cakey.pem
-rw-r--r-- 1 tiian tiian 1424 2016-03-30 21:57 cacert.pem
-rw-r--r-- 1 tiian tiian  963 2016-03-30 21:57 private/cakey.pem
~~~

### Creating the first X.509 certificate
To implement a *mutual authentication* configuration a distinct certificate for every node is necessary.

#### Retrieving node/system unique ID ####
FLoM uses unique IDs provided by [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/).

Creating a unique ID with FLoM is like using the command *dbus-uuidgen \-\-get*:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ flom --unique-id
91ed6d1ed76c5773c7503d285679b33b
~~~

From the above text, the **unique ID** of node1 is *91ed6d1ed76c5773c7503d285679b33b*.    
The *unique ID* must be passed to the *openssl* command when the *Common Name* associated to the certificate is asked.

Now you have to execute 4 commands:

* the *unique ID* must be passed to the first command
* the third command asks for a password: the same password used for the certification authority (see above).

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl req -nodes -new -x509 -keyout node1_key.pem -out node1_req.pem -days 3650 -config flom_openssl.conf
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl x509 -x509toreq -in node1_req.pem -signkey node1_key.pem -out tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl ca -config flom_openssl.conf -policy policy_anything -out node1_cert.pem -infiles tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA2$ rm tmp.pem
~~~

The output should be something like this:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl req -nodes -new -x509 -keyout node1_key.pem -out node1_req.pem -days 3650 -config flom_openssl.conf
Generating a 1024 bit RSA private key
...................................++++++
........................................................................................................++++++
writing new private key to 'node1_key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IT]:
State or Province Name (full name) [Treviso]:
Locality Name (eg, city) [Mogliano Veneto]:
Organization Name (eg, company) [FLoM Software Corporation]:
Organizational Unit Name (eg, section) [Development and Research]:
Common Name (eg, YOUR name) []:91ed6d1ed76c5773c7503d285679b33b
Email Address []:
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl x509 -x509toreq -in node1_req.pem -signkey node1_key.pem -out tmp.pem
Getting request Private Key
Generating certificate request
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl ca -config flom_openssl.conf -policy policy_anything -out node1_cert.pem -infiles tmp.pem
Using configuration from flom_openssl.conf
Enter pass phrase for ./private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Mar 30 20:12:38 2016 GMT
            Not After : Mar 30 20:12:38 2017 GMT
        Subject:
            countryName               = IT
            stateOrProvinceName       = Treviso
            localityName              = Mogliano Veneto
            organizationName          = FLoM Software Corporation
            organizationalUnitName    = Development and Research
            commonName                = 91ed6d1ed76c5773c7503d285679b33b
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                2A:A7:12:B2:7C:DC:B7:93:A9:89:C9:F1:1A:A7:6B:86:9D:99:12:7E
            X509v3 Authority Key Identifier: 
                keyid:B3:44:2B:D2:40:4D:FB:89:E5:F0:FF:A3:20:8A:F3:F5:9C:C5:89:A9

Certificate is to be certified until Mar 30 20:12:38 2017 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
tiian@ubuntu1004:~/flom_ssl/CA2$ rm tmp.pem
~~~

If everything is fine, you must have two files: *node1_cert.pem* contains the **X.509 certificate** for *node1* (*ubuntu1004*) and *node1_key.pem* contains the **private key** associated to the certicate:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la node1_cert.pem node1_key.pem
-rw-r--r-- 1 tiian tiian 3433 2016-03-30 22:12 node1_cert.pem
-rw-r--r-- 1 tiian tiian  887 2016-03-30 22:11 node1_key.pem
~~~

### Creating the second X.509 certificate
To implement a *mutual authentication* configuration a distinct certificate for every node is necessary.

Connect to *node2* and retrieve the *unique ID*:

~~~
[tiian@centos71-64 ~]$ flom --unique-id
6046574205df4258aeb409bf377235e0
~~~

From the above text, the **unique ID** of node2 is *6046574205df4258aeb409bf377235e0*.   
The *unique ID* must be passed to the *openssl* command when the *Common Name* associated to the certificate is asked.

Connect to *node1* and execute the 4 commands you need to generate the certificate for *node2*:

* the *unique ID* must be passed to the first command
* the third command asks for a password: the same password used for the certification authority (see above).

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl req -nodes -new -x509 -keyout node2_key.pem -out node2_req.pem -days 3650 -config flom_openssl.conf
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl x509 -x509toreq -in node2_req.pem -signkey node2_key.pem -out tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl ca -config flom_openssl.conf -policy policy_anything -out node2_cert.pem -infiles tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA2$ rm tmp.pem
~~~

The output should be something like this:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl req -nodes -new -x509 -keyout node2_key.pem -out node2_req.pem -days 3650 -config flom_openssl.conf
Generating a 1024 bit RSA private key
....................................................................++++++
...++++++
writing new private key to 'node2_key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IT]:
State or Province Name (full name) [Treviso]:
Locality Name (eg, city) [Mogliano Veneto]:
Organization Name (eg, company) [FLoM Software Corporation]:
Organizational Unit Name (eg, section) [Development and Research]:
Common Name (eg, YOUR name) []:6046574205df4258aeb409bf377235e0
Email Address []:
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl x509 -x509toreq -in node2_req.pem -signkey node2_key.pem -out tmp.pem
Getting request Private Key
Generating certificate request
tiian@ubuntu1004:~/flom_ssl/CA2$ openssl ca -config flom_openssl.conf -policy policy_anything -out node2_cert.pem -infiles tmp.pem
Using configuration from flom_openssl.conf
Enter pass phrase for ./private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 2 (0x2)
        Validity
            Not Before: Mar 30 20:21:44 2016 GMT
            Not After : Mar 30 20:21:44 2017 GMT
        Subject:
            countryName               = IT
            stateOrProvinceName       = Treviso
            localityName              = Mogliano Veneto
            organizationName          = FLoM Software Corporation
            organizationalUnitName    = Development and Research
            commonName                = 6046574205df4258aeb409bf377235e0
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                D0:54:C2:2B:69:A1:DD:F1:61:0C:E0:DD:82:71:2B:50:73:9F:B4:0E
            X509v3 Authority Key Identifier: 
                keyid:B3:44:2B:D2:40:4D:FB:89:E5:F0:FF:A3:20:8A:F3:F5:9C:C5:89:A9

Certificate is to be certified until Mar 30 20:21:44 2017 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
tiian@ubuntu1004:~/flom_ssl/CA2$ rm tmp.pem
~~~

If everything is fine, you must have two new files: *node2_cert.pem* contains the **X.509 certificate** for *node2* (*centos71-64*) and *node2_key.pem* contains the **private key** associated to the certicate:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la node2_cert.pem node2_key.pem
-rw-r--r-- 1 tiian tiian 3433 2016-03-30 22:21 node2_cert.pem
-rw-r--r-- 1 tiian tiian  891 2016-03-30 22:21 node2_key.pem
~~~

### Certificate *"installation"* ###
Only 3 files are needed to FLoM process (*flom*):

* *node?_cert.pem*
* *node?_key.pem*
* *cacert.pem*

copy them in a *easy to use place* on both systems.     
Pay attention you have to copy different certificate and key to different systems.     
Local copy:

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ mkdir /tmp/flom_ssl
tiian@ubuntu1004:~/flom_ssl/CA2$ cp cacert.pem node1_cert.pem node1_key.pem /tmp/flom_ssl/
tiian@ubuntu1004:~/flom_ssl/CA2$ ls -la /tmp/flom_ssl/
total 20
drwxr-xr-x 2 tiian tiian 4096 2016-03-30 22:25 .
drwxrwxrwt 5 root  root  4096 2016-03-30 22:25 ..
-rw-r--r-- 1 tiian tiian 1424 2016-03-30 22:25 cacert.pem
-rw-r--r-- 1 tiian tiian 3433 2016-03-30 22:25 node1_cert.pem
-rw-r--r-- 1 tiian tiian  887 2016-03-30 22:25 node1_key.pem
~~~

Remote copy (directory */tmp/flom_ssl/* on node2 must be created in advance):

~~~
tiian@ubuntu1004:~/flom_ssl/CA2$ scp -r cacert.pem node2_cert.pem node2_key.pem tiian@centos71-64.brenta.org:/tmp/flom_ssl/
Enter passphrase for key '/home/tiian/.ssh/id_rsa': 
cacert.pem                                    100% 1424     1.4KB/s   00:00    
node2_cert.pem                                100% 3433     3.4KB/s   00:00    
node2_key.pem                                 100%  891     0.9KB/s   00:00    
~~~

Check the content in node2:

~~~
[tiian@centos71-64 ~]$ ls -la /tmp/flom_ssl/
total 16
drwxrwxr-x.  2 tiian tiian   64 30 mar 22.31 .
drwxrwxrwt. 10 root  root  4096 30 mar 22.31 ..
-rw-r--r--.  1 tiian tiian 1424 30 mar 22.31 cacert.pem
-rw-r--r--.  1 tiian tiian 3433 30 mar 22.31 node2_cert.pem
-rw-r--r--.  1 tiian tiian  891 30 mar 22.31 node2_key.pem
~~~

### Debugging TLS (mutual authentication security level) with FLoM
Setting a *trace mask* to trace the messaged produced by *flom_tls*, *flom_tcp* and *flom_debug* modules can help to troubleshoot a possible issue.

[[img src=security_ma_01.png]]

This is the command to start a FLoM debug server using TLS inside node1:

~~~
tiian@ubuntu1004:~$ export FLOM_TRACE_MASK=0x70000
tiian@ubuntu1004:~$ echo $FLOM_TRACE_MASK0x70000
tiian@ubuntu1004:~$ flom --debug-feature=tls.server -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/node1_cert.pem --tls-private-key=/tmp/flom_ssl/node1_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=yes
~~~

This is the command to start a FLoM client using TLS inside node2:

~~~
[tiian@centos71-64 ~]$ export FLOM_TRACE_MASK=0x70000
[tiian@centos71-64 ~]$ echo $FLOM_TRACE_MASK0x70000
[tiian@centos71-64 ~]$ flom --debug-feature=tls.client -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/node2_cert.pem --tls-private-key=/tmp/flom_ssl/node2_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=yes
~~~

This is the output that you should obtain on node1 (debug server):

~~~
2016-04-01 22:58:06.368658 [1869/0x13a4510] flom_debug_features
2016-04-01 22:58:06.368709 [1869/0x13a4510] flom_debug_features: name='tls.server'
2016-04-01 22:58:06.368715 [1869/0x13a4510] flom_debug_features_tls_server
2016-04-01 22:58:06.368724 [1869/0x13a4510] flom_tcp_init
2016-04-01 22:58:06.368728 [1869/0x13a4510] flom_tcp_init
2016-04-01 22:58:06.368734 [1869/0x13a4510] flom_tls_init: calling SSL_library_init()...
2016-04-01 22:58:06.368803 [1869/0x13a4510] flom_tls_init: calling SSL_load_error_strings()...
2016-04-01 22:58:06.369261 [1869/0x13a4510] flom_tls_init: calling OpenSSL_add_all_algorithms()...
2016-04-01 22:58:06.369330 [1869/0x13a4510] flom_tls_context
2016-04-01 22:58:06.369335 [1869/0x13a4510] flom_tls_context: setting TLS/SSL method to TLSv1_server_method()
2016-04-01 22:58:06.369473 [1869/0x13a4510] flom_tls_context: SSL_CTX_set_verify(0x13cb830, 3, flom_tls_callback)
2016-04-01 22:58:06.369483 [1869/0x13a4510] flom_tls_context/excp=2/ret_cod=0/errno=2
2016-04-01 22:58:06.369489 [1869/0x13a4510] flom_tls_set_cert
2016-04-01 22:58:06.369492 [1869/0x13a4510] flom_tls_set_cert: SSL_CTX_use_certificate_file(obj->ctx, '/tmp/flom_ssl/node1_cert.pem', SSL_FILETYPE_PEM)
2016-04-01 22:58:06.369616 [1869/0x13a4510] flom_tls_set_cert: SSL_CTX_use_PrivateKey_file(obj->ctx, '/tmp/flom_ssl/node1_key.pem', SSL_FILETYPE_PEM)
2016-04-01 22:58:06.369661 [1869/0x13a4510] flom_tls_set_cert: SSL_CTX_check_private_key(obj->ctx)
2016-04-01 22:58:06.369669 [1869/0x13a4510] flom_tls_set_cert: SSL_CTX_load_verify_locations(obj->ctx, '/tmp/flom_ssl/cacert.pem', NULL)
2016-04-01 22:58:06.369736 [1869/0x13a4510] flom_tls_set_cert/excp=4/ret_cod=0/errno=2
2016-04-01 22:58:06.369744 [1869/0x13a4510] flom_tcp_listen
2016-04-01 22:58:06.369747 [1869/0x13a4510] flom_tcp_listen: binding address 'ubuntu1004.brenta.org' and port 28015
2016-04-01 22:58:06.370755 [1869/0x13a4510] flom_tcp_listen/getaddrinfo(): [ai_flags=1,ai_family=2,ai_socktype=1,ai_protocol=6,ai_addrlen=16,ai_canonname='{null}'] [ai_flags=1,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='{null}'] 
2016-04-01 22:58:06.370773 [1869/0x13a4510] flom_tcp_listen: ai_addr addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:58:06.370797 [1869/0x13a4510] flom_tcp_listen: bound!
2016-04-01 22:58:06.370808 [1869/0x13a4510] flom_tcp_listen/excp=3/ret_cod=0/errno=22
2016-04-01 22:58:21.619398 [1869/0x13a4510] flom_debug_features_tls_server: incoming connection address data: addrlen=16; IPv4 address, sin_port=56578, sin_addr='192.168.122.12'
2016-04-01 22:58:21.619451 [1869/0x13a4510] flom_tls_accept
2016-04-01 22:58:21.619461 [1869/0x13a4510] flom_tls_prepare
2016-04-01 22:58:21.619553 [1869/0x13a4510] flom_tls_prepare/excp=3/ret_cod=0/errno=22
...
2016-04-01 22:58:21.624687 [1869/0x13a4510] flom_tls_accepted: connection accepted with AES256-SHA encryption
2016-04-01 22:58:21.624703 [1869/0x13a4510] flom_tls_cert_parse
...
2016-04-01 22:58:21.624785 [1869/0x13a4510] flom_tls_cert_parse: issuer fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Mutual Authentication
...
2016-04-01 22:58:21.625015 [1869/0x13a4510] flom_tls_cert_parse: subject fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=6046574205df4258aeb409bf377235e0
2016-04-01 22:58:21.625063 [1869/0x13a4510] flom_tls_cert_parse/excp=4/ret_cod=0/errno=0
2016-04-01 22:58:21.625072 [1869/0x13a4510] flom_tls_accept/excp=3/ret_cod=0/errno=0
2016-04-01 22:58:21.625079 [1869/0x13a4510] flom_tls_recv
2016-04-01 22:58:21.625715 [1869/0x13a4510] flom_tls_recv: received 32 of 100 bytes
2016-04-01 22:58:21.625725 [1869/0x13a4510] flom_tls_recv/excp=1/ret_cod=0/errno=0
2016-04-01 22:58:21.625731 [1869/0x13a4510] flom_debug_features_tls_server: received 32 bytes, '6046574205df4258aeb409bf377235e0'
2016-04-01 22:58:21.625756 [1869/0x13a4510] flom_tls_cert_check
2016-04-01 22:58:21.625761 [1869/0x13a4510] flom_tls_cert_check: peer address='192.168.122.12/56578', CN='6046574205df4258aeb409bf377235e0', peer unique ID='6046574205df4258aeb409bf377235e0'
2016-04-01 22:58:21.625838 [1869/0x13a4510] flom_tls_cert_check/excp=2/ret_cod=0/errno=0
2016-04-01 22:58:21.625913 [1869/0x13a4510] flom_debug_features_tls_server: sending 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
2016-04-01 22:58:21.625921 [1869/0x13a4510] flom_tls_send
2016-04-01 22:58:21.625963 [1869/0x13a4510] flom_tls_send/excp=2/ret_cod=0/errno=0
2016-04-01 22:58:21.625974 [1869/0x13a4510] flom_tcp_close
2016-04-01 22:58:21.625990 [1869/0x13a4510] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:58:21.625996 [1869/0x13a4510] flom_tcp_close
2016-04-01 22:58:21.626007 [1869/0x13a4510] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:58:21.626115 [1869/0x13a4510] flom_debug_features_tls_server/excp=13/ret_cod=0/errno=0
2016-04-01 22:58:21.626123 [1869/0x13a4510] flom_debug_features/excp=1/ret_cod=0/errno=0
~~~

This is the output that you should obtain on node2 (debug client):

~~~
2016-04-01 22:58:20.961363 [2080/0x1cc3200] flom_debug_features
2016-04-01 22:58:20.961459 [2080/0x1cc3200] flom_debug_features: name='tls.client'
2016-04-01 22:58:20.961472 [2080/0x1cc3200] flom_debug_features_tls_client
2016-04-01 22:58:20.961481 [2080/0x1cc3200] flom_tcp_init
2016-04-01 22:58:20.961490 [2080/0x1cc3200] flom_tls_init: calling SSL_library_init()...
2016-04-01 22:58:20.961662 [2080/0x1cc3200] flom_tls_init: calling SSL_load_error_strings()...
2016-04-01 22:58:20.964320 [2080/0x1cc3200] flom_tls_init: calling OpenSSL_add_all_algorithms()...
2016-04-01 22:58:20.964463 [2080/0x1cc3200] flom_tls_context
2016-04-01 22:58:20.964470 [2080/0x1cc3200] flom_tls_context: setting TLS/SSL method to TLSv1_client_method()
2016-04-01 22:58:20.964754 [2080/0x1cc3200] flom_tls_context: SSL_CTX_set_verify(0x1ce1bd0, 1, flom_tls_callback)
2016-04-01 22:58:20.964771 [2080/0x1cc3200] flom_tls_context/excp=2/ret_cod=0/errno=2
2016-04-01 22:58:20.964778 [2080/0x1cc3200] flom_tls_set_cert
2016-04-01 22:58:20.964783 [2080/0x1cc3200] flom_tls_set_cert: SSL_CTX_use_certificate_file(obj->ctx, '/tmp/flom_ssl/node2_cert.pem', SSL_FILETYPE_PEM)
2016-04-01 22:58:20.965069 [2080/0x1cc3200] flom_tls_set_cert: SSL_CTX_use_PrivateKey_file(obj->ctx, '/tmp/flom_ssl/node2_key.pem', SSL_FILETYPE_PEM)
2016-04-01 22:58:20.965163 [2080/0x1cc3200] flom_tls_set_cert: SSL_CTX_check_private_key(obj->ctx)
2016-04-01 22:58:20.965178 [2080/0x1cc3200] flom_tls_set_cert: SSL_CTX_load_verify_locations(obj->ctx, '/tmp/flom_ssl/cacert.pem', NULL)
2016-04-01 22:58:20.965331 [2080/0x1cc3200] flom_tls_set_cert/excp=4/ret_cod=0/errno=2
2016-04-01 22:58:20.965345 [2080/0x1cc3200] flom_tcp_connect
2016-04-01 22:58:20.965349 [2080/0x1cc3200] flom_tcp_connect: connecting to address 'ubuntu1004.brenta.org' and port 28015
2016-04-01 22:58:20.966482 [2080/0x1cc3200] flom_tcp_connect/getaddrinfo(): [ai_flags=2,ai_family=2,ai_socktype=1,ai_protocol=6,ai_addrlen=16,ai_canonname='ubuntu1004.brenta.org'] [ai_flags=2,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='{null}'] 
2016-04-01 22:58:20.966513 [2080/0x1cc3200] flom_tcp_try_connect: sa addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:58:20.966556 [2080/0x1cc3200] flom_tcp_try_connect: sa addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:58:20.966983 [2080/0x1cc3200] flom_tcp_connect: domain=2, sockfd=3, socket_type=16, addrlen=0
2016-04-01 22:58:20.967007 [2080/0x1cc3200] flom_tcp_connect: addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:58:20.967031 [2080/0x1cc3200] flom_tcp_connect/excp=2/ret_cod=0/errno=22
2016-04-01 22:58:20.967038 [2080/0x1cc3200] flom_tls_connect
2016-04-01 22:58:20.967044 [2080/0x1cc3200] flom_tls_prepare
2016-04-01 22:58:20.967081 [2080/0x1cc3200] flom_tls_prepare/excp=3/ret_cod=0/errno=22
...
2016-04-01 22:58:20.972668 [2080/0x1cc3200] flom_tls_connect: connection established with AES256-SHA encryption
2016-04-01 22:58:20.972689 [2080/0x1cc3200] flom_tls_cert_parse
...
2016-04-01 22:58:20.972763 [2080/0x1cc3200] flom_tls_cert_parse: issuer fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Mutual Authentication
...
2016-04-01 22:58:20.972945 [2080/0x1cc3200] flom_tls_cert_parse: subject fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=91ed6d1ed76c5773c7503d285679b33b
2016-04-01 22:58:20.972966 [2080/0x1cc3200] flom_tls_cert_parse/excp=4/ret_cod=0/errno=0
2016-04-01 22:58:20.972971 [2080/0x1cc3200] flom_tls_connect/excp=3/ret_cod=0/errno=0
2016-04-01 22:58:20.973278 [2080/0x1cc3200] flom_debug_features_tls_client: sending 32 bytes, '6046574205df4258aeb409bf377235e0'
2016-04-01 22:58:20.973289 [2080/0x1cc3200] flom_tls_send
2016-04-01 22:58:20.973325 [2080/0x1cc3200] flom_tls_send/excp=2/ret_cod=0/errno=0
2016-04-01 22:58:20.973338 [2080/0x1cc3200] flom_tls_recv
2016-04-01 22:58:20.973845 [2080/0x1cc3200] flom_tls_recv: received 32 of 100 bytes
2016-04-01 22:58:20.973860 [2080/0x1cc3200] flom_tls_recv/excp=1/ret_cod=0/errno=0
2016-04-01 22:58:20.973866 [2080/0x1cc3200] flom_debug_features_tls_client: received 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
2016-04-01 22:58:20.973887 [2080/0x1cc3200] flom_tls_cert_check
2016-04-01 22:58:20.973892 [2080/0x1cc3200] flom_tls_cert_check: peer address='192.168.122.57/28015', CN='91ed6d1ed76c5773c7503d285679b33b', peer unique ID='91ed6d1ed76c5773c7503d285679b33b'
2016-04-01 22:58:20.973913 [2080/0x1cc3200] flom_tls_cert_check/excp=2/ret_cod=0/errno=0
2016-04-01 22:58:20.973919 [2080/0x1cc3200] flom_tcp_close
2016-04-01 22:58:20.973952 [2080/0x1cc3200] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:58:20.974059 [2080/0x1cc3200] flom_debug_features_tls_client/excp=10/ret_cod=0/errno=0
2016-04-01 22:58:20.974067 [2080/0x1cc3200] flom_debug_features/excp=1/ret_cod=0/errno=0
~~~

Remove *trace mask*, then restart the debug server on node1:

~~~
tiian@ubuntu1004:~$ unset FLOM_TRACE_MASK
tiian@ubuntu1004:~$ flom --debug-feature=tls.server -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/node1_cert.pem --tls-private-key=/tmp/flom_ssl/node1_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=yes
tiian@ubuntu1004:~$ echo $?
0
~~~

Remove *trace mask*, then restart the client on node2:

~~~
[tiian@centos71-64 ~]$ unset FLOM_TRACE_MASK
[tiian@centos71-64 ~]$ flom --debug-feature=tls.client -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/node2_cert.pem --tls-private-key=/tmp/flom_ssl/node2_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=yes
[tiian@centos71-64 ~]$ echo $?
0
~~~

### Configuration hint
A more convenient way to setup all the TLS parameters is to use a FLoM [Configuration](../Configuration.md) file: these are the keys you have to customize:

~~~
[TLS]
# Name of the file that contains the X.509 certificate assigned to this peer
# (Uncomment below row if necessary)
#TlsCertificate=cert.pem
# Name of the file that contains the private key of this peer
# (Uncomment below row if necessary)
#TlsPrivateKey=priv_key.pem
# Name of the file that contains the X.509 certificate of the certification
# authority used to sign the certificate of this peer
# (Uncomment below row if necessary)
#TlsCaCertificate=ca_cert.pem
# Check if the CommonName (CN) of the peer certificate matches the peer unique
# identifier; valid values are "yes" and "no" (case insensitive)
# (Uncomment below row if necessary)
#TlsCheckPeerId=yes
~~~

### System message logging

Both client and server writes logging messages on the system log.

These are the messages produced on syslog by the debug server:

~~~
tiian@ubuntu1004:~$ sudo tail /var/log/syslog 
Apr  1 22:58:21 ubuntu1004 flom: FLM011I X.509 CA certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Mutual Authentication
Apr  1 22:58:21 ubuntu1004 flom: FLM012I X.509 peer certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=6046574205df4258aeb409bf377235e0
Apr  1 22:58:21 ubuntu1004 flom: FLM014I peer '192.168.122.12/56578' with unique ID '6046574205df4258aeb409bf377235e0' was authenticated using CN field '6046574205df4258aeb409bf377235e0' inside the presented X.509 certificate
~~~

These are the messages produced on syslog by the debug client:

~~~
[tiian@centos71-64 ~]$ sudo tail /var/log/messages
Apr  1 22:58:20 centos71-64 flom: FLM011I X.509 CA certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Mutual Authentication
Apr  1 22:58:20 centos71-64 flom: FLM012I X.509 peer certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=91ed6d1ed76c5773c7503d285679b33b
Apr  1 22:58:20 centos71-64 flom: FLM014I peer '192.168.122.57/28015' with unique ID '91ed6d1ed76c5773c7503d285679b33b' was authenticated using CN field '91ed6d1ed76c5773c7503d285679b33b' inside the presented X.509 certificate
~~~

