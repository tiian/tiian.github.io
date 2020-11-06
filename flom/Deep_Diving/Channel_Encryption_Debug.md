# Channel Encryption Debugging
This debug session uses two different nodes; these are the names used during all the session:

* node1, name: *ubuntu1004.brenta.org*, operating system: Ubuntu Linux 10.04 (64 bit)
* node2, name: *centos71-64.brenta.org*, operating system: CentOS 7.1 (64 bit)

## Native FLoM debugging tool
Sometimes debugging becomes difficult, especially when you have to deal with networks, firewalls, public and private addresses and so on.   
For some complex features, FLoM provide a native debugging tool that should help system engineers to understand the root cause of an issue.

## Setting up the certification authority and the required certificates

### Creating a CA (Certification Authority) of name *CA1*
Connect to node1, and execute these commands to create the directory structure:

~~~
tiian@ubuntu1004:~$ mkdir flom_ssl
tiian@ubuntu1004:~$ cd flom_ssl
tiian@ubuntu1004:~/flom_ssl$ mkdir CA1
tiian@ubuntu1004:~/flom_ssl$ cd CA1
tiian@ubuntu1004:~/flom_ssl/CA1$ mkdir certs crl newcerts private
tiian@ubuntu1004:~/flom_ssl/CA1$ echo "01" > serial
tiian@ubuntu1004:~/flom_ssl/CA1$ cp /dev/null index.txt
tiian@ubuntu1004:~/flom_ssl/CA1$ ls -la
total 28
drwxr-xr-x 6 tiian tiian 4096 2016-03-29 22:09 .
drwxr-xr-x 3 tiian tiian 4096 2016-03-29 22:09 ..
drwxr-xr-x 2 tiian tiian 4096 2016-03-29 22:09 certs
drwxr-xr-x 2 tiian tiian 4096 2016-03-29 22:09 crl
-rw-r--r-- 1 tiian tiian    0 2016-03-29 22:09 index.txt
drwxr-xr-x 2 tiian tiian 4096 2016-03-29 22:09 newcerts
drwxr-xr-x 2 tiian tiian 4096 2016-03-29 22:09 private
-rw-r--r-- 1 tiian tiian    3 2016-03-29 22:09 serial
~~~

Pick-up a *openssl.cnf* example file; FLoM provide a pre-configured file in directory */usr/local/share/doc/flom/*:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ ls -la /usr/local/share/doc/flom/flom_openssl.conf 
-rw-r--r-- 1 root root 9431 2016-03-28 19:34 /usr/local/share/doc/flom/flom_openssl.conf
~~~

copy it locally:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ cp /usr/local/share/doc/flom/flom_openssl.conf .
~~~

Generate the certificate for the CA:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -days 3650 -config flom_openssl.conf 
Generating a 1024 bit RSA private key
...++++++
..............++++++
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
Common Name (eg, YOUR name) []:CA for FLoM Channel Encryption
Email Address []:
~~~

File *cacert.pem* contains the **X.509 certificate** of the certification authority you have just created and file *private/cakey.pem* contains the **private key** associated to the certification authority.

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ ls -la cacert.pem private/cakey.pem 
-rw-r--r-- 1 tiian tiian 1411 2016-03-29 22:19 cacert.pem
-rw-r--r-- 1 tiian tiian  963 2016-03-29 22:19 private/cakey.pem
~~~

### Creating a first X.509 certificate ###
To implement a *channel encryption* configuration just one certificate is enough.

Now you have to execute 4 commands; the system asks for a password: the same password used for the certification authority (see above).

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl req -nodes -new -x509 -keyout first_key.pem -out first_req.pem -days 3650 -config flom_openssl.conf
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl x509 -x509toreq -in first_req.pem -signkey first_key.pem -out tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl ca -config flom_openssl.conf -policy policy_anything -out first_cert.pem -infiles tmp.pem
tiian@ubuntu1004:~/flom_ssl/CA1$ rm tmp.pem
~~~

The output should be something like this:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl req -nodes -new -x509 -keyout first_key.pem -out first_req.pem -days 3650 -config flom_openssl.conf
Generating a 1024 bit RSA private key
.........++++++
...++++++
writing new private key to 'first_key.pem'
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
Common Name (eg, YOUR name) []:Generic FLoM node
Email Address []:
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl x509 -x509toreq -in first_req.pem -signkey first_key.pem -out tmp.pem
Getting request Private Key
Generating certificate request
tiian@ubuntu1004:~/flom_ssl/CA1$ openssl ca -config flom_openssl.conf -policy policy_anything -out first_cert.pem -infiles tmp.pem
Using configuration from flom_openssl.conf
Enter pass phrase for ./private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Mar 29 20:29:07 2016 GMT
            Not After : Mar 29 20:29:07 2017 GMT
        Subject:
            countryName               = IT
            stateOrProvinceName       = Treviso
            localityName              = Mogliano Veneto
            organizationName          = FLoM Software Corporation
            organizationalUnitName    = Development and Research
            commonName                = Generic FLoM node
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                E9:77:80:9F:6F:A7:1D:55:E6:46:31:48:91:E8:64:DD:37:3B:58:5D
            X509v3 Authority Key Identifier: 
                keyid:71:E4:77:AE:FD:4B:17:9C:4D:9E:7C:B6:1D:8D:37:08:F2:DD:09:AC

Certificate is to be certified until Mar 29 20:29:07 2017 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
tiian@ubuntu1004:~/flom_ssl/CA1$ rm tmp.pem
~~~

If everything is fine, you must have two files: *first_cert.pem* contains the **X.509 certificate** for your FLoM node(s) and *first_key.pem* contains the **private key** associated to the certicate:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ ls -la first_cert.pem first_key.pem 
-rw-r--r-- 1 tiian tiian 3391 2016-03-29 22:29 first_cert.pem
-rw-r--r-- 1 tiian tiian  887 2016-03-29 22:26 first_key.pem
~~~

### Certificate *"installation"* ###
Only 3 files are needed to FLoM process (*flom*):

* *first_cert.pem*
* *first_key.pem*
* *cacert.pem*

copy them in a *easy to use place* on both systems.
Local copy:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ mkdir /tmp/flom_ssl
tiian@ubuntu1004:~/flom_ssl/CA1$ cp cacert.pem first_cert.pem first_key.pem /tmp/flom_ssl/
tiian@ubuntu1004:~/flom_ssl/CA1$ ls -la /tmp/flom_ssl/
total 20
drwxr-xr-x 2 tiian tiian 4096 2016-03-29 22:39 .
drwxrwxrwt 5 root  root  4096 2016-03-29 22:39 ..
-rw-r--r-- 1 tiian tiian 1411 2016-03-29 22:39 cacert.pem
-rw-r--r-- 1 tiian tiian 3391 2016-03-29 22:39 first_cert.pem
-rw-r--r-- 1 tiian tiian  887 2016-03-29 22:39 first_key.pem
~~~

Remote copy:

~~~
tiian@ubuntu1004:~/flom_ssl/CA1$ scp -r /tmp/flom_ssl/ tiian@centos71-64.brenta.org:/tmp
Enter passphrase for key '/home/tiian/.ssh/id_rsa': 
first_cert.pem                                100% 3391     3.3KB/s   00:00    
cacert.pem                                    100% 1411     1.4KB/s   00:00    
first_key.pem                                 100%  887     0.9KB/s   00:00    
~~~

Check the content in node2:

~~~
[tiian@centos71-64 CA1]$ ls -la /tmp/flom_ssl/
total 16
drwxr-xr-x.  2 tiian tiian   64 29 mar 22.40 .
drwxrwxrwt. 10 root  root  4096 29 mar 22.40 ..
-rw-r--r--.  1 tiian tiian 1411 29 mar 22.40 cacert.pem
-rw-r--r--.  1 tiian tiian 3391 29 mar 22.40 first_cert.pem
-rw-r--r--.  1 tiian tiian  887 29 mar 22.40 first_key.pem
~~~

### Debugging TLS (channel encryption security level) with FLoM
Setting a *trace mask* to trace the messages produced by *flom_tls*, *flom_tcp* and *flom_debug* modules can help to troubleshoot a possible issue.

![](security_ce_01.png)

This is the command to start a FLoM debug server using TLS inside node1:

~~~
tiian@ubuntu1004:~$ export FLOM_TRACE_MASK=0x270000
tiian@ubuntu1004:~$ echo $FLOM_TRACE_MASK
0x270000
tiian@ubuntu1004:~$ flom --debug-feature=tls.server -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/first_cert.pem --tls-private-key=/tmp/flom_ssl/first_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=no
~~~

This is the command to start a FLoM debug client using TLS inside node2:

~~~
[tiian@centos71-64 ~]$ export FLOM_TRACE_MASK=0x70000[tiian@centos71-64 ~]$ echo $FLOM_TRACE_MASK0x70000
[tiian@centos71-64 ~]$ flom --debug-feature=tls.client -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/first_cert.pem --tls-private-key=/tmp/flom_ssl/first_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=no
~~~

This is the output that you should obtain on node1 (debug server):

~~~
2016-04-01 22:22:30.172686 [1348/0xfec510] flom_debug_features
2016-04-01 22:22:30.172813 [1348/0xfec510] flom_debug_features: name='tls.server'
2016-04-01 22:22:30.172829 [1348/0xfec510] flom_debug_features_tls_server
2016-04-01 22:22:30.172850 [1348/0xfec510] flom_tcp_init
2016-04-01 22:22:30.172860 [1348/0xfec510] flom_tcp_init
2016-04-01 22:22:30.172872 [1348/0xfec510] flom_tls_init: calling SSL_library_init()...
2016-04-01 22:22:30.173009 [1348/0xfec510] flom_tls_init: calling SSL_load_error_strings()...
2016-04-01 22:22:30.174172 [1348/0xfec510] flom_tls_init: calling OpenSSL_add_all_algorithms()...
2016-04-01 22:22:30.174338 [1348/0xfec510] flom_tls_context
2016-04-01 22:22:30.174349 [1348/0xfec510] flom_tls_context: setting TLS/SSL method to TLSv1_server_method()
2016-04-01 22:22:30.174693 [1348/0xfec510] flom_tls_context: SSL_CTX_set_verify(0x1013830, 3, flom_tls_callback)
2016-04-01 22:22:30.174717 [1348/0xfec510] flom_tls_context/excp=2/ret_cod=0/errno=2
2016-04-01 22:22:30.174732 [1348/0xfec510] flom_tls_set_cert
2016-04-01 22:22:30.174740 [1348/0xfec510] flom_tls_set_cert: SSL_CTX_use_certificate_file(obj->ctx, '/tmp/flom_ssl/first_cert.pem', SSL_FILETYPE_PEM)
2016-04-01 22:22:30.175043 [1348/0xfec510] flom_tls_set_cert: SSL_CTX_use_PrivateKey_file(obj->ctx, '/tmp/flom_ssl/first_key.pem', SSL_FILETYPE_PEM)
2016-04-01 22:22:30.175215 [1348/0xfec510] flom_tls_set_cert: SSL_CTX_check_private_key(obj->ctx)
2016-04-01 22:22:30.175243 [1348/0xfec510] flom_tls_set_cert: SSL_CTX_load_verify_locations(obj->ctx, '/tmp/flom_ssl/cacert.pem', NULL)
2016-04-01 22:22:30.175424 [1348/0xfec510] flom_tls_set_cert/excp=4/ret_cod=0/errno=2
2016-04-01 22:22:30.175445 [1348/0xfec510] flom_tcp_listen
2016-04-01 22:22:30.175454 [1348/0xfec510] flom_tcp_listen: binding address 'ubuntu1004.brenta.org' and port 28015
2016-04-01 22:22:30.177735 [1348/0xfec510] flom_tcp_listen/getaddrinfo(): [ai_flags=1,ai_family=2,ai_socktype=1,ai_protocol=6,ai_addrlen=16,ai_canonname='{null}'] [ai_flags=1,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='{null}'] 
2016-04-01 22:22:30.177781 [1348/0xfec510] flom_tcp_listen: ai_addr addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:22:30.177840 [1348/0xfec510] flom_tcp_listen: bound!
2016-04-01 22:22:30.177868 [1348/0xfec510] flom_tcp_listen/excp=3/ret_cod=0/errno=22
2016-04-01 22:22:46.170112 [1348/0xfec510] flom_debug_features_tls_server: incoming connection address data: addrlen=16; IPv4 address, sin_port=56573, sin_addr='192.168.122.12'
2016-04-01 22:22:46.170179 [1348/0xfec510] flom_tls_accept
2016-04-01 22:22:46.170190 [1348/0xfec510] flom_tls_prepare
2016-04-01 22:22:46.170252 [1348/0xfec510] flom_tls_prepare/excp=3/ret_cod=0/errno=22
2016-04-01 22:22:46.174396 [1348/0xfec510] flom_tls_callback: preverify_ok=1
2016-04-01 22:22:46.174418 [1348/0xfec510] flom_tls_callback: ret_cod=1
2016-04-01 22:22:46.174592 [1348/0xfec510] flom_tls_callback: preverify_ok=1
2016-04-01 22:22:46.174601 [1348/0xfec510] flom_tls_callback: ret_cod=1
2016-04-01 22:22:46.176158 [1348/0xfec510] flom_tls_accepted: connection accepted with AES256-SHA encryption
2016-04-01 22:22:46.176175 [1348/0xfec510] flom_tls_cert_parse
...
2016-04-01 22:22:46.176257 [1348/0xfec510] flom_tls_cert_parse: issuer fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Channel Encryption
...
2016-04-01 22:22:46.176489 [1348/0xfec510] flom_tls_cert_parse: subject fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=Generic FLoM node
2016-04-01 22:22:46.176536 [1348/0xfec510] flom_tls_cert_parse/excp=4/ret_cod=0/errno=0
2016-04-01 22:22:46.176545 [1348/0xfec510] flom_tls_accept/excp=3/ret_cod=0/errno=0
2016-04-01 22:22:46.176552 [1348/0xfec510] flom_tls_recv
2016-04-01 22:22:46.178669 [1348/0xfec510] flom_tls_recv: received 32 of 100 bytes
2016-04-01 22:22:46.178687 [1348/0xfec510] flom_tls_recv/excp=1/ret_cod=0/errno=0
2016-04-01 22:22:46.178694 [1348/0xfec510] flom_debug_features_tls_server: received 32 bytes, '6046574205df4258aeb409bf377235e0'
2016-04-01 22:22:46.178771 [1348/0xfec510] flom_debug_features_tls_server: sending 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
2016-04-01 22:22:46.178780 [1348/0xfec510] flom_tls_send
2016-04-01 22:22:46.178824 [1348/0xfec510] flom_tls_send/excp=2/ret_cod=0/errno=0
2016-04-01 22:22:46.178836 [1348/0xfec510] flom_tcp_close
2016-04-01 22:22:46.178878 [1348/0xfec510] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:22:46.178890 [1348/0xfec510] flom_tcp_close
2016-04-01 22:22:46.178940 [1348/0xfec510] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:22:46.179087 [1348/0xfec510] flom_debug_features_tls_server/excp=13/ret_cod=0/errno=0
2016-04-01 22:22:46.179101 [1348/0xfec510] flom_debug_features/excp=1/ret_cod=0/errno=0
tiian@ubuntu1004:~$ echo $?
0
~~~

This is the output that you should obtain on node2 (debug client):

~~~
2016-04-01 22:22:45.506324 [2010/0x256d200] flom_debug_features
2016-04-01 22:22:45.506557 [2010/0x256d200] flom_debug_features: name='tls.client'
2016-04-01 22:22:45.506575 [2010/0x256d200] flom_debug_features_tls_client
2016-04-01 22:22:45.506589 [2010/0x256d200] flom_tcp_init
2016-04-01 22:22:45.506602 [2010/0x256d200] flom_tls_init: calling SSL_library_init()...
2016-04-01 22:22:45.506892 [2010/0x256d200] flom_tls_init: calling SSL_load_error_strings()...
2016-04-01 22:22:45.510156 [2010/0x256d200] flom_tls_init: calling OpenSSL_add_all_algorithms()...
2016-04-01 22:22:45.510326 [2010/0x256d200] flom_tls_context
2016-04-01 22:22:45.510333 [2010/0x256d200] flom_tls_context: setting TLS/SSL method to TLSv1_client_method()
2016-04-01 22:22:45.510830 [2010/0x256d200] flom_tls_context: SSL_CTX_set_verify(0x258bbd0, 1, flom_tls_callback)
2016-04-01 22:22:45.510854 [2010/0x256d200] flom_tls_context/excp=2/ret_cod=0/errno=2
2016-04-01 22:22:45.510863 [2010/0x256d200] flom_tls_set_cert
2016-04-01 22:22:45.510869 [2010/0x256d200] flom_tls_set_cert: SSL_CTX_use_certificate_file(obj->ctx, '/tmp/flom_ssl/first_cert.pem', SSL_FILETYPE_PEM)
2016-04-01 22:22:45.511231 [2010/0x256d200] flom_tls_set_cert: SSL_CTX_use_PrivateKey_file(obj->ctx, '/tmp/flom_ssl/first_key.pem', SSL_FILETYPE_PEM)
2016-04-01 22:22:45.511355 [2010/0x256d200] flom_tls_set_cert: SSL_CTX_check_private_key(obj->ctx)
2016-04-01 22:22:45.511372 [2010/0x256d200] flom_tls_set_cert: SSL_CTX_load_verify_locations(obj->ctx, '/tmp/flom_ssl/cacert.pem', NULL)
2016-04-01 22:22:45.511568 [2010/0x256d200] flom_tls_set_cert/excp=4/ret_cod=0/errno=2
2016-04-01 22:22:45.511585 [2010/0x256d200] flom_tcp_connect
2016-04-01 22:22:45.511591 [2010/0x256d200] flom_tcp_connect: connecting to address 'ubuntu1004.brenta.org' and port 28015
2016-04-01 22:22:45.515958 [2010/0x256d200] flom_tcp_connect/getaddrinfo(): [ai_flags=2,ai_family=2,ai_socktype=1,ai_protocol=6,ai_addrlen=16,ai_canonname='ubuntu1004.brenta.org'] [ai_flags=2,ai_family=10,ai_socktype=1,ai_protocol=6,ai_addrlen=28,ai_canonname='{null}'] 
2016-04-01 22:22:45.515999 [2010/0x256d200] flom_tcp_try_connect: sa addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:22:45.517012 [2010/0x256d200] flom_tcp_try_connect: sa addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:22:45.517670 [2010/0x256d200] flom_tcp_connect: domain=2, sockfd=3, socket_type=16, addrlen=0
2016-04-01 22:22:45.517701 [2010/0x256d200] flom_tcp_connect: addrlen=16; IPv4 address, sin_port=28015, sin_addr='192.168.122.57'
2016-04-01 22:22:45.517733 [2010/0x256d200] flom_tcp_connect/excp=2/ret_cod=0/errno=22
2016-04-01 22:22:45.517743 [2010/0x256d200] flom_tls_connect
2016-04-01 22:22:45.517750 [2010/0x256d200] flom_tls_prepare
2016-04-01 22:22:45.517842 [2010/0x256d200] flom_tls_prepare/excp=3/ret_cod=0/errno=22
2016-04-01 22:22:45.519259 [2010/0x256d200] flom_tls_callback: preverify_ok=1
2016-04-01 22:22:45.519286 [2010/0x256d200] flom_tls_callback: ret_cod=1
2016-04-01 22:22:45.519449 [2010/0x256d200] flom_tls_callback: preverify_ok=1
2016-04-01 22:22:45.519459 [2010/0x256d200] flom_tls_callback: ret_cod=1
2016-04-01 22:22:45.524159 [2010/0x256d200] flom_tls_connect: connection established with AES256-SHA encryption
2016-04-01 22:22:45.524179 [2010/0x256d200] flom_tls_cert_parse
...
2016-04-01 22:22:45.524251 [2010/0x256d200] flom_tls_cert_parse: issuer fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Channel Encryption
...
2016-04-01 22:22:45.525551 [2010/0x256d200] flom_tls_cert_parse: subject fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=Generic FLoM node
2016-04-01 22:22:45.525573 [2010/0x256d200] flom_tls_cert_parse/excp=4/ret_cod=0/errno=0
2016-04-01 22:22:45.525578 [2010/0x256d200] flom_tls_connect/excp=3/ret_cod=0/errno=0
2016-04-01 22:22:45.526165 [2010/0x256d200] flom_debug_features_tls_client: sending 32 bytes, '6046574205df4258aeb409bf377235e0'
2016-04-01 22:22:45.526177 [2010/0x256d200] flom_tls_send
2016-04-01 22:22:45.526214 [2010/0x256d200] flom_tls_send/excp=2/ret_cod=0/errno=0
2016-04-01 22:22:45.526223 [2010/0x256d200] flom_tls_recv
2016-04-01 22:22:45.526721 [2010/0x256d200] flom_tls_recv: received 32 of 100 bytes
2016-04-01 22:22:45.526734 [2010/0x256d200] flom_tls_recv/excp=1/ret_cod=0/errno=0
2016-04-01 22:22:45.526739 [2010/0x256d200] flom_debug_features_tls_client: received 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
2016-04-01 22:22:45.526746 [2010/0x256d200] flom_tcp_close
2016-04-01 22:22:45.526771 [2010/0x256d200] flom_tcp_close/excp=1/ret_cod=0/errno=0
2016-04-01 22:22:45.526951 [2010/0x256d200] flom_debug_features_tls_client/excp=10/ret_cod=0/errno=0
2016-04-01 22:22:45.526966 [2010/0x256d200] flom_debug_features/excp=1/ret_cod=0/errno=0
[tiian@centos71-64 ~]$ echo $?
0
~~~

#### The key messages are the following ones:

The debug client send its unique ID:
~~~
2016-04-01 22:22:45.526165 [2010/0x256d200] flom_debug_features_tls_client: sending 32 bytes, '6046574205df4258aeb409bf377235e0'
~~~

the debug server receives the unique ID sent by the debug client and replies with its own unique ID:

~~~
2016-04-01 22:22:46.178694 [1348/0xfec510] flom_debug_features_tls_server: received 32 bytes, '6046574205df4258aeb409bf377235e0'
2016-04-01 22:22:46.178771 [1348/0xfec510] flom_debug_features_tls_server: sending 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
~~~

the debug client receives the unique ID sent by the debug server:

~~~
2016-04-01 22:22:45.526739 [2010/0x256d200] flom_debug_features_tls_client: received 32 bytes, '91ed6d1ed76c5773c7503d285679b33b'
~~~

Remove *trace mask* , then restart the debug server on node1:

~~~
tiian@ubuntu1004:~$ unset FLOM_TRACE_MASK
tiian@ubuntu1004:~$ flom --debug-feature=tls.server -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/first_cert.pem --tls-private-key=/tmp/flom_ssl/first_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=no
tiian@ubuntu1004:~$ echo $?
0
~~~

Remove *trace mask*, then restart the debug client on node2:

~~~
[tiian@centos71-64 ~]$ unset FLOM_TRACE_MASK
[tiian@centos71-64 ~]$ flom --debug-feature=tls.client -a ubuntu1004.brenta.org --tls-certificate=/tmp/flom_ssl/first_cert.pem --tls-private-key=/tmp/flom_ssl/first_key.pem --tls-ca-certificate=/tmp/flom_ssl/cacert.pem --tls-check-peer-id=no
[tiian@centos71-64 ~]$ echo $?
0
~~~

When everything is fine, no messages on the terminal are printed and the exit code of the *flom* command is 0.

### Configuration hint
A more convenient way to setup all the TLS parameters is to use a FLoM [configuration](../Configuration.md) file: these are the keys you have to customize:

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

Both client and server write logging messages on the system log.

On the server side (*node1*):

~~~
tiian@ubuntu1004:~$ sudo tail /var/log/syslog 
Apr  1 22:22:46 ubuntu1004 flom: FLM011I X.509 CA certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Channel Encryption
Apr  1 22:22:46 ubuntu1004 flom: FLM012I X.509 peer certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=Generic FLoM node
Apr  1 22:32:50 ubuntu1004 flom: FLM011I X.509 CA certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Channel Encryption
Apr  1 22:32:50 ubuntu1004 flom: FLM012I X.509 peer certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=Generic FLoM node
~~~

On the client side (*node2*), these are the corresponding messages:

~~~
[tiian@centos71-64 ~]$ sudo tail /var/log/messages
Apr  1 22:32:49 centos71-64 flom: FLM011I X.509 CA certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=CA for FLoM Channel Encryption
Apr  1 22:32:49 centos71-64 flom: FLM012I X.509 peer certificate fields are C=IT/ST=Treviso/L=Mogliano Veneto/O=FLoM Software Corporation/OU=Development and Research/emailAddress={null}/CN=Generic FLoM node
~~~

