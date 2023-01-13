# Identity Service JWKS Endpoint for Doccla

> The JSON Web Key Set (JWKS) endpoint is a read-only endpoint that contains the public key information
> in the JWKS format. The public keys are the counterpart of the private keys which is used to sign the tokens.

## JWKS Endpoints

| KID            | Environment    | JWKS Endpoint                                                                                |
| -------------- | -------------- | -------------------------------------------------------------------------------------------- |
| prs-prod-1     | Production     | https://raw.githubusercontent.com/doccla/identity-service-jwks/main/production/jwks.json     |
| prs-non-prod-1 | Non-Production | https://raw.githubusercontent.com/doccla/identity-service-jwks/main/non-production/jwks.json |

## Generate Private/Public Key Pair

To generate key pairs for production or test environment, follow these steps:

1. Open your terminal, ensure you have a BASH shell.
2. Set an environment variables:
   ```bash
   KID=YOUR_KID_NAME # Example KID=prs-prod-1
   ```
3. Execute the following commands:
   ```bash
   openssl genrsa -out $KID.pem 4096
   openssl rsa -in $KID.pem -pubout -outform PEM -out $KID.pem.pub
   ```

This generates the public and private keys. (**NOTE: DO NOT SHARE OR UPLOAD THE PUBLIC & PRIVATE KEY, KEEP IN 1Password Vault.**)

## Generate JWKS JSON

To generate the JWKS file for prodcution or test environment, follow these steps:

1. Open your terminal, esnure you have a BASH shell.
2. Execute the following command to get the "modulus" of the private key:
   ```bash
   MODULUS=$(
    openssl rsa -pubin -in $KID.pem.pub -noout -modulus `# Print modulus of public key` \
        | cut -d '=' -f2                                    `# Extract modulus value from output` \
        | xxd -r -p                                         `# Convert from string to bytes` \
        | openssl base64 -A                                 `# Base64 encode without wrapping lines` \
        | sed 's|+|-|g; s|/|_|g; s|=||g'                    `# URL encode as JWK standard requires`
   )
   ```
3. Next execute the following command to generate the JWSK file (using the RS512 algorithm):
   ```bash
   echo '{
        "keys": [
            {
                "kty": "RSA",
                "n": "'"$MODULUS"'",
                "e": "AQAB",
                "alg": "RS512",
                "kid": "'"$KID"'",
                "use": "sig"
            }
        ]
    }' > $KID.json
   ```

Copy the content of the `Keys` content and add/update the `production/jwks.json` file, the file is a JSON object, therefore you can have multiple JWKS in 1 file.
