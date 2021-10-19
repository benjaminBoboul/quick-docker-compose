KEY_PASSWORD:=node_pass
KEYSTORE_PASSWORD:=node_pass
TRUSTSTORE_PASSWORD:=node_pass
KEY_SIZE=4096

.PHONY: clear

key.creds:
	echo ${KEY_PASSWORD} > $@

ca.key:
	openssl genrsa -out ca.key ${KEY_SIZE}

ca.crt: ca.key
	openssl req -new -x509 -sha256 -key ca.key -out ca.crt -subj /CN=MyProject/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR -days 9999	

%.key:
	openssl genrsa -out $@ ${KEY_SIZE}

%.pem: %.key
	openssl pkcs8 -topk8 -in $< -inform PEM -v1 PBE-SHA1-3DES -out $@ -outform PEM -nocrypt 

%.csr: %.pem
	openssl req -new -key $< -out $@ -subj /CN=$(*F)/OU=MyUnit/O=Organism/L=SomeCity/ST=Somewhere/C=FR

%.crt: %.csr ca.crt ca.key
	openssl x509 -req -in $< -CA ca.crt -CAkey ca.key -days 9999 -CAcreateserial -out $@

required_files: admin.crt admin.pem ca.crt

clear:
	find ./ -type f \( -name \*.csr -o -name \*.srl -o -name \*.crt -o -name \*.key -o -name \*.pem -o -name \*.p12 -o -name \*.creds \) -delete