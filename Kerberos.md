# AD Server

## Account
Create an account on the Windows AD Server to act as the delegate account for Kerberos authentication.

1. Open Active Directory Users and Computers
2. Right Click on User -> Create New User
3. When creating the user set password to never expire
4. Right click on new user and select properties
5. Go to the account tab -> account options and set the following flags to true:
    * This account supports Kerberos AES 256 bit encryption
6. Click Ok

## SPN
Note: SPNs must be unique, i.e. a user may have multiple SPNs but an SPN should map to only 1 user.

## Delegation
Note: The delegation tab will not be available until an SPN for the user has been created.


# Glassfish

## SPNEGO

### Installation

1. Go to the lib directory under glassfish
2. Download the spnego.jar from sourceforge (https://sourceforge.net/projects/spnego/files/)
3. rename jar to spnego.jar

```
cd glassfish/lib
wget https://ufpr.dl.sourceforge.net/project/spnego/spnego-r7.jar
mv spnego-r7.jar spnego.jar
```

### Configuration

The following configs can be found at 
```
cd glassfish/domains/domain1/config
```

where ```domain1``` is the name of your domain, replace as necessary

#### default-web.confg

```
nano glassfish/domains/domain1/config/default-web.xml
```

Add the following lines

```xml
<!-- Create a filter for SPNEGO -->
<filter>
    <filter-name>SpnegoHttpFilter</filter-name>
    <filter-class>net.sourceforge.spnego.SpnegoHttpFilter</filter-class>

    <init-param>
        <param-name>spnego.allow.basic</param-name>
        <param-value>true</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.allow.localhost</param-name>
        <param-value>true</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.allow.unsecure.basic</param-name>
        <param-value>true</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.login.client.module</param-name>
        <param-value>spnego-client</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.krb5.conf</param-name>
        <param-value>krb5.conf</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.login.conf</param-name>
        <param-value>login.conf</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.preauth.username</param-name>
        <param-value>glassfish</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.preauth.password</param-name>
        <param-value>pass</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.login.server.module</param-name>
        <param-value>spnego-server</param-value>
    </init-param>

   <init-param>
        <param-name>spnego.prompt.ntlm</param-name>
        <param-value>true</param-value>
    </init-param>

    <init-param>
        <param-name>spnego.logger.level</param-name>
        <param-value>1</param-value>
    </init-param>
</filter>

<!-- Filter all traffic through our spnego filter -->
<filter-mapping>
    <filter-name>SpnegoHttpFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

Replace ```spnego.preauth.username``` with the user name that was created in AD for delegation purposes
Replace ```spnego.preauth.password``` with the password for said user

**NOTE: when setting this up in a production environment use a keytab file instead, that way the username and password won't be stored in plain text on the glassfish server.**

#### krb5.conf

```
nano glassfish/domains/domain1/config/krb5.conf
```

Add the following lines:
```
[libdefaults]
        default_realm = DOMAIN.LOCAL
        default_tkt_enctypes = rc4-hmac aes256-cts-hmac-sha1-96 aes128-cts
        default_tgs_enctypes = rc4-hmac aes256-cts-hmac-sha1-96 aes128-cts
        permitted_enctypes   = rc4-hmac aes256-cts-hmac-sha1-96 aes128-cts

[realms]
        DOMAIN.LOCAL  = {
                kdc = win-dc5dtuhlf9n.DOMAIN.LOCAL
                default_domain = DOMAIN.LOCAL
}

[domain_realm]
        .DOMAIN.LOCAL = DOMAIN.LOCAL
```
- ```DOMAIN.LOCAL``` is the name of the realm, replace as necessary
- ```win-dc5dtuhlf9n``` is the name of the KDC server, replace as necessary.

If you do not know the name of your KDC server, go to your AD machine and run the following in command prompt to retrieve the name.

```
nslookup -type=srv _kerberos._tcp.DOMAIN.LOCAL
```
- ```DOMAIN.LOCAL``` is the name of the realm, replace as necessary

A response like this will be returned
```
Server:  UnKnown
Address:  192.168.1.24

_kerberos._tcp.QUACK.TEST       SRV service location:
          priority       = 0
          weight         = 100
          port           = 88
          svr hostname   = win-dc5dtuhlf9n.quack.test
win-dc5dtuhlf9n.quack.test      internet address = 192.168.1.24
```
the ```svr hostname``` defines your KDC server

Note, in windows 10 the browsers will communicate via ```aes256-cts-hmac-sha1-96``` encryption

#### login.conf

```
nano glassfish/domains/domain1/config/login.conf
```

Add the following lines:
```
spnego-client {
        com.sun.security.auth.module.Krb5LoginModule required;
};
spnego-server {
        com.sun.security.auth.module.Krb5LoginModule required
        storeKey=true
        isInitiator=false;
};
```

- **storekey**: Set this to true to if you want the principal's key to be stored in the Subject's private credentials. 
- **isInitiator**: Set this to true, if initiator. Set this to false, if acceptor only. (Default is true). Note: Do not set this value to false for initiators.

For futher details on the Krb5LoginModule parameters see: https://docs.oracle.com/javase/7/docs/jre/api/security/jaas/spec/com/sun/security/auth/module/Krb5LoginModule.html



# Tools

## klist
## kinit

# Troubleshooting
## hash doesn't match?
try running kinit on the webserver to clear any outdated tickets???

# References.
1. http://spnego.sourceforge.net/spnego_glassfish.html
2. https://docs.bmc.com/docs/decisionsupportserverautomation/85/locating-active-directory-kdcs-350325160.html
3. https://stackoverflow.com/questions/3686420/servlet-filter-url-mapping
4. https://docs.oracle.com/javase/7/docs/jre/api/security/jaas/spec/com/sun/security/auth/module/Krb5LoginModule.html