# SSL Encryption 

## Basics of SSL
[What is SSL?](https://www.cloudflare.com/learning/ssl/what-is-ssl/)

![ssl](https://media.geeksforgeeks.org/wp-content/uploads/20250801143329769220/what_is_ssl_certificate_.webp)

- SSL stands for Secure Sockets Layer.
- It is an older security protocol used to encrypt data transmitted over the Internet.
- TLS (Transport Layer Security) is the newer and more secure successor to SSL.
- SSL is now deprecated because it has known security vulnerabilities.
- Today, websites use TLS, although people still commonly refer to it as “SSL.”
- HTTPS means that HTTP is protected using TLS.
- TLS provides:
    - Data encryption
    - Authentication of the website
    - Data integrity, preventing information from being altered in transit

![ca](https://www.hypr.com/hubfs/Media%20Graphics/ca.png)

## SSL in Kafka
- Encrypting data between brokers and clients
- Kafka client will trust the certificate of the Kafka Broker and they can securely exchange encrypted data
- SSL in Kafka also can be used for authentication
- Kafka Broker in SSL mode works on Port 9093 by default

Steps in Kafka:
1. Create a Certificate Authority (CA) -> receive cert and key (private key)
2. Setup brokers certificates
3. Sign broker certificates
4. Setup a key store for brokers
5. Setup a trust strore for clients
6. Reboot brokers in SSL mode (Port 9093)
7. Test setup using secure SSL producer and consumer