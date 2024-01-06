
# Jobwave installation

## Pre-configuration

First, you need to create the RSA keypair in order for JWT to be generated and verified.

```shell
cd docker/data
# Generate the private key without password
openssl genrsa -out private.pem 2048
# Extract the public key from the private
openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```

Then, you will need to create then own the directories that cannot be owned by the rootless containers themselves (i.e. all Bitnami containers).

```shell
# If not done already, move to the data folder
cd docker/data
# Create the directories (if absent)
mkdir -p users/mongo
# Change the owner of said directories
chown 1000:1000 -R users/mongo
```

### Running

To run the program, simply execute the following command in the root folder :

```shell
docker compose up -d
```

The gateway should be accessible through `localhost:8080`.
