# Requirements: Certificate Generation (Self-Signed)

## Overview

Prepare for using HTTPS with a self-signed certificate (you act as the
certificate authority). Review this module's resources on how CAs enable HTTPS
in production. Then generate a self-signed certificate with Java Keytool and
document the outputs.

## Rubric Criteria

- Certificate Authorities: Investigate and explain the role and value of CA
  services.
  - Why would you want to use a CA for security?
  - What are the advantages of using a CA?
- Certificate Generation: Use Java Keytool to generate a self-signed
  certificate (you are the CA).
  - Ensure Java/JDK is installed so `keytool` is available (no extra install).
  - Find your Java home to locate `keytool`/`keytool.exe`.

## Commands to Run (update the password values)

Generate a 2048-bit RSA key pair and self-signed certificate (valid 360 days):

```sh
keytool -genkey -keyalg RSA -alias selfsigned -keypass <password> \
  -keystore keystore.jks -storepass <password> -validity 360 -keysize 2048
```

Export the certificate to `server.cer`:

```sh
keytool -export -alias selfsigned -storepass <password> \
  -file server.cer -keystore keystore.jks
```

Print the certificate details from `server.cer`:

```sh
keytool -printcert -file server.cer
```

Notes:

- Replace `<password>` with a unique, secure password (used consistently for
  keypass and storepass).
- On Windows, the command may appear as `keytool.exe ...`; syntax is otherwise
  the same.

## Evidence to Capture

- Screenshot of answering the keytool prompts (distinguished name questions).
- Screenshot of the certificate printout (`keytool -printcert -file server.cer`)
  showing owner, issuer, serial, validity dates, fingerprints, signature
  algorithm, public key algorithm, version, and extensions.
