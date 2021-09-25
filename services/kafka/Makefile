CA_PASSWORD:=ca_pass
KEY_PASSWORD:=node_pass
KEYSTORE_PASSWORD:=node_pass
TRUSTSTORE_PASSWORD:=node_pass
KEY_SIZE=4096

.PHONY: clear

keystore.creds:
	echo ${KEYSTORE_PASSWORD} > keystore.creds

truststore.creds:
	echo ${TRUSTSTORE_PASSWORD} > keystore.creds

ca.key ca.crt:
	openssl genrsa -out ca.key --passout pass:${CA_PASSWORD} ${KEY_SIZE}
	openssl req -new -x509 -key ca.key -out ca.crt -sha256 -subj /CN=MyProject/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR -days 9999	

truststore.p12: ca.crt
    # Generate a PKCS#12 truststore from ca certificate
	openssl pkcs12 -export -nokeys -in $< -out $@ -passout pass:${TRUSTSTORE_PASSWORD}

%.key:
	openssl genrsa -out $@ ${KEY_SIZE}

%.csr %.crt: %.key ca.crt ca.key
	@echo "--> Generate certificate for $*:"
	openssl req -new -key $< -out $*.csr -subj /CN=$(*F)/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR
	openssl x509 -req -CA ca.crt -CAkey ca.key -in $*.csr -out $*.crt -days 9999 -CAcreateserial

%.pem: %.key
    # Convert to PKCS#8 for kafka
	openssl pkcs8 -topk8 -in $< -inform pem -v1 PBE-SHA1-RC4-128 -out $@ -outform pem -passout pass:${KEY_PASSWORD}

keystore-%.p12: %.pem %.crt ca.crt
    # Merge certificates & key into a PKCS#12 keystore
	openssl pkcs12 -export -inkey $*.pem -in $*.crt -out $@ -name $(*F) -CAfile ca.crt -caname MyProject -passin pass:${KEY_PASSWORD} -passout pass:${KEYSTORE_PASSWORD}

clear:
	find ./ -type f \( -name \*.csr -o -name \*.srl -o -name \*.crt -o -name \*.key -o -name \*.pem \) -delete