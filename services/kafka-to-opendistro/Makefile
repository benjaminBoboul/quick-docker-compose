CA_PASSWORD:=changeit
KEY_PASSWORD:=changeit
KEYSTORE_PASSWORD:=changeit
TRUSTSTORE_PASSWORD:=changeit
KEY_SIZE=4096

.PHONY: clear


default: keystore-kafka.jks truststore.jks admin.crt admin.pem key.creds keystore.creds truststore.creds


key.creds:
	echo ${KEY_PASSWORD} > $@

keystore.creds:
	echo ${KEYSTORE_PASSWORD} > $@

truststore.creds:
	echo ${TRUSTSTORE_PASSWORD} > $@




ca.key:
	openssl genrsa -out $@ --passout pass:${CA_PASSWORD} ${KEY_SIZE}

ca.crt: ca.key
	openssl req -new -x509 -key $< -out $@ -sha256 -subj /CN=MyProject/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR -days 9999	

truststore.p12: ca.crt
    # Generate a PKCS#12 truststore from ca certificate
	openssl pkcs12 -export -nokeys -in $< -out $@ -passout pass:${TRUSTSTORE_PASSWORD}

truststore.jks: ca.crt
    # Generate JKS archive using keytool utility
	keytool -noprompt -keystore $@ -alias cacert -import -file $<  -storepass ${TRUSTSTORE_PASSWORD}




%.key:
	openssl genrsa -out $@ ${KEY_SIZE}

%.pem: %.key
    # Convert to PKCS#8 with PKCS#12 compatible algorithm
	openssl pkcs8 -nocrypt -topk8 -in $< -inform pem -v1 PBE-SHA1-3DES -out $@ -outform pem

keystore-%.pem: %.key
    # Convert to PKCS#8 with PKCS#12 compatible algorithm
	openssl pkcs8 -topk8 -in $< -inform pem -v1 PBE-SHA1-RC4-128 -out $@ -outform pem -passout pass:${KEY_PASSWORD}

%.csr: %.key
    # Generate CSR based on PEM key
	openssl req -new -key $< -out $@ -subj /CN=$(*F)/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR 

keystore-%.csr: keystore-%.pem
    # Generate CSR based on PEM key
	openssl req -new -key $< -out $@ -subj /CN=$(*F)/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR -passin pass:${KEY_PASSWORD}

%.crt: %.csr ca.crt ca.key
	openssl x509 -req -in $< -CA ca.crt -CAkey ca.key -days 9999 -CAcreateserial -out $@

%.p12: %.pem %.crt
    # Create PKCS12 keystore from private key and public certificate.
	openssl pkcs12 -export -name $(*F) -in ${@:.p12=.crt} -inkey ${@:.p12=.pem} -passin pass:${KEY_PASSWORD} -out $@ -passout pass:${KEYSTORE_PASSWORD}

%.jks: %.p12
	keytool -importkeystore -destkeystore $@ -srckeystore $< -srcstoretype pkcs12 -alias $(*F) -srcstorepass ${KEYSTORE_PASSWORD} -deststorepass ${KEYSTORE_PASSWORD}




clear:
	find ./ -type f \( -name \*.csr -o -name \*.srl -o -name \*.crt -o -name \*.key -o -name \*.pem -o -name \*.jks -o -name \*.creds \) -delete