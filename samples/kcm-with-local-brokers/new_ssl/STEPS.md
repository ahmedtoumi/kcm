- Générer la nouvelle autorité de certification (CA)
> openssl genrsa -out ca.key 2048
- Cela créera une nouvelle clé privée (ca.key) et un certificat racine auto-signé (ca.crt).
> openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt \
-subj "/C=FR/ST=State/L=City/O=Company/OU=Dev/CN=MyCA"

- Générer un certificat serveur pour un broker Kafka (par exemple, broker-1)
  Utilisons cette nouvelle CA pour signer un certificat serveur. Créez d'abord une clé privée et une demande de signature de certificat (CSR) :
> openssl genrsa -out broker-1.key 2048

> openssl req -new -key broker-1.key -out broker-1.csr \
-subj "/C=FR/ST=State/L=City/O=Company/OU=Dev/CN=broker-1"

- Ensuite, signez ce CSR avec votre CA :
> openssl x509 -req -in broker-1.csr -CA ca.crt -CAkey ca.key \
-CAcreateserial -out broker-1.crt -days 365 -sha256

- Créer un keystore et un truststore
  Importez la clé privée et le certificat dans un keystore :
> openssl pkcs12 -export -in broker-1.crt -inkey broker-1.key -out broker-1.p12 -name broker-1 \
-passout pass:changeit

- Puis convertissez ce fichier PKCS12 en un keystore Java :
> keytool -importkeystore -deststorepass changeit -destkeypass changeit \
-destkeystore kafka.keystore.jks -srckeystore broker-2.p12 \
-srcstoretype PKCS12 -srcstorepass changeit -alias broker-2

- Ajoutez le certificat CA au truststore :
> keytool -import -trustcacerts -alias ca -file ca.crt \
-keystore kafka.truststore.jks -storepass changeit -noprompt
