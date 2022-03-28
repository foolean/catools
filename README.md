# catools - Certificate Authority Toolkit

catools is a collection of scripts that simplifies the operation
of a certificate authority (CA) while following generally accepted
best practices.


## Certificate Authorities

    +--+ root
       |
       +--+ intermediate
          |
          +--+ device
          +--+ identity
          +--+ puppet

### root

The root CA is the top of the certificate food chain, the holy grail.  It's
only purpose is to create the intermediate CA and revoke it in the event of
dire circumstances.

The catools aliases default to placing the root CA inside its own encrypted
container (LUKS).  It would not be uncommon for the Security admin to burn
the root CA to a CD and store it in a secure location such as a safe.


### intermediate

The intermediate CA is used to create the lower level CAs.  It's purpose is
provide another layer of protection within the CA infrastructure.  It also
allows for the intermediate CA to be resigned by the root CA thus allowing
for the rest of the CAs to be extended in near perpetuity.

The catools aliases default to placing the intermediate CA inside its own
encrypted container (LUKS).  It would not be uncommon for the Security admin
to burn the intermediate CA, along with the root CA, to a CD and store it in
a secure location such as a safe.


### device

The device CA is used to create device specific certificates.  These will be
client and server certificates.


### identity

The identity CA is used to create user specific certificates.  These will be
identity (email) and encryption certificates.  The encryption certificates
are for encryption only where the identity certificate is for signing as well
as encryption.

In most cases, users will only need an identity certificate


### puppet

The puppet CA is used to allow Puppet infrastructure to use a true CA instead
if the typical self-signed certificates.


## Certificate types

### device

  * client

    # Example for a single client
    `create_client_cert -d -v -n "node1.yourdomain.tld"`

    # Example for a client that appears as part of a farm
    `create_client_cert -d -v -s "DNS:pool.yourdomain.tld" -n "node1.yourdomain.tld"`

  * server

    # Example for a single web server
    `create_server_cert -d -v -n "www.yourdomain.tld"`

    # Example for a web server that is part of a farm
    `create_server_cert -d -v -s "DNS:www.yourdomain.tld" -n "www1.yourdomain.tld"`


### identity

  * email

    `create_identity_cert -d -v -n "John Q. Public" -e "jqp@yourdomain.tld`

  * encryption

    `create_encryption_cert -d -v -n "John Q. Public" -e "jqp@yourdomain.tld`


### Download catools

    git clone https://github.com/foolean/catools.git


## Setting up your CA

It is recommended that you place the entire CA structure inside an encrypted
container for further protection.   You can use lukstools to create the
container and then copy the contents of "extra" into the root of your CA
directory.   Sourcing the initial .ca file will mount the CA structure and
also source the .ca file within.

    1. Create the root of your CA
       a. create a 'ca' and 'enc' directory

    2. Copy 'bin' and '.ca' from <catools>

    3. Create the encrypted container

        `bin/lukstools -s "<container_size>" -c "enc/data.enc" -k -f -d -v`

    4. Source the .ca file

        `source .ca`

    5. Mount the CA

        `mount_ca`

    6. Clone catools inside your encrypted container

        `git clone https://github.com/foolean/catools.git ca`


### The quick way

1. Configure <catools>/etc/ca.conf

2. Load the catools environment

    `source <catools>/.ca`

3. Create the entire CA structure

    `create_full_ca`


### The manual way

1. Configure <catools>/etc/ca.conf

2. Load the catools environment

    `source <catools>/.ca`

3. Create your CA GPG key

    `create_gpg_key -d -v`

4. Create your root CA

    `create_ca -d -v -n root`

5. Create your intermediate CA

    `create_ca -d -v -p root -n intermediate`

6. Create your device CA

    `create_ca -d -v -p intermediate -n device`

7. Create your identity CA

    `create_ca -d -v -p intermediate -n identity`

6. Create your puppet CA

    `create_ca -d -v -p intermediate -n puppet`


## References

https://buildmedia.readthedocs.org/media/pdf/pki-tutorial/latest/pki-tutorial.pdf

