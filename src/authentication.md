# Authentication 
Authentication in Kafka:
- SSL Authentication -> client authenticate to Kafka using SSL certificates
- SASL Authentication -> supported protocols:
    - PLAIN -> username and password
    - SCRAM -> username and password
    - GSSAPI -> Kerberos / Active Directory
    - OAUTHBEARER
    - LDAP/KEYCLOAK?

## SSL

![2](https://docs.oracle.com/cd/E19575-01/819-3669/images/security-sslBMAWithCertificates.gif)

## SASL Kerberos 
[Kerberos - Geeks for Geeks](https://www.geeksforgeeks.org/computer-networks/kerberos/)

- Kerberos is a protocol for authentication over unsecure networks
- Data exchange is encrypted
- Based on tickets issued to Service Principals (also a user is a service in kerberos)
- Involves a trusted 3rd-party (KDC - Key Distribution Center)
- Created at MIT
- Microsoft Active DIrectory is the most popular implementation of Kerberos
- The free implementation is krb5

![2](https://miro.medium.com/v2/1*AT6ELO6T9LspAJINz9EOHw.png)