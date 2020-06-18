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

## Adding a machine to the AD (For Devs)
When adding a server to the AD the DNS on that machine must be set to the IP address of the AD machine in order for it to resolve workstation names.

### Windows
#### Setting DNS Details:
1. Open Control Panel\Network and Internet\Network and Sharing Center
2. Select Change Adapter Settings
3. Right Click on the relevant adapter and select properties
4. Select Internet Protocol Version 4
5. Click Properties
6. Select User the following DNS Server addresses
7. Set the DNS to the IP of the AD machine
8. Select Ok on all open dialogs.

#### Adding a Server to AD:
1. Go to Control Panel\System and Security\System
2. Select Advanced System Settings
3. Select the Computer Name tab and select the change button
4. Set Member of to Domain and add the domain name
5. Select ok on all open dialogs and restart the machine as necessary

### Linux
#### Setting DNS Details:

```
sudo nano /etc/resolv.conf
```

Add the following lines

```
nameserver 192.168.1.24
```
where 192.168.1.24 is the IP address of your AD machine

#### Adding a Server to AD:
install the following
``` 
sudo apt-get install sssd realmd samba-common-bin samba-dsdb-modules adcli
```

run the following command to add to AD
```
realm join --install=/ --user=administrator DOMAIN.LOCAL --verbose
```
where ```administrator``` is an administrator in the AD and ```DOMAIN.LOCAL``` is the realm

# Glassfish

## krb5-user
This will add the kinit and klist commands to a linux (debian) machine

```
sudo apt-get update
sudo apt-get install krb5-user
```

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


# Workstation

In order for a browser in windows to send the Kerberos token to a server the following changes must be made.

1. Go to internet options -> security -> local intranet -> sites
2. Click the Advanced button
3. Add the root url of the glassfish server to the zone. e.g. http://win-d9l9n7ep4fs
4. Click close then click okay
5. Select the Advanced tab
6. Scroll to the security section at the bottom
7. Ensure "Enable Integrated Windows Authentication" is checked, if not check it.
8. Click Okay to close the dialog. (restart the workstation if you changed "Enable Integrated Windows Authentication")

If doing this for all workstations on the AD, make sure these changes are set in group policy and propagated down.

# Tools

## klist
## kinit

# Troubleshooting

## krb5 checksum failed
try running kinit on the webserver to clear any outdated tickets???

# References.
1. http://spnego.sourceforge.net/spnego_glassfish.html
2. https://docs.bmc.com/docs/decisionsupportserverautomation/85/locating-active-directory-kdcs-350325160.html
3. https://stackoverflow.com/questions/3686420/servlet-filter-url-mapping
4. https://docs.oracle.com/javase/7/docs/jre/api/security/jaas/spec/com/sun/security/auth/module/Krb5LoginModule.html
5. https://support.rackspace.com/how-to/changing-dns-settings-on-linux/


https://community.spiceworks.com/how_to/144319-join-debian-to-ad

https://lists.freedesktop.org/archives/authentication/2016-April/000343.html

https://www.raspberrypi.org/forums/viewtopic.php?t=98948