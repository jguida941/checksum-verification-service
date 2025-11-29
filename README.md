# Checksum Verification & Certificate Toolkit

Spring Boot sample that generates a checksum for a unique string using a secure
message digest algorithm, plus accompanying docs for generating self-signed
certificates with Java Keytool.

## Docs

- Requirements and rubric:
  [doc/requirements/checkSum-Verification/requirements.md](doc/requirements/checkSum-Verification/requirements.md)
- Certificate generation requirements:
  [doc/requirements/certificate-generation/requirements.md](doc/requirements/certificate-generation/requirements.md)
- Submission template:
  [doc/templates/ChecksumVerificationTemplate.md](doc/templates/ChecksumVerificationTemplate.md)
- Protecting sensitive data reading:
  [doc/readings/ProtectingSensitiveData.md](doc/readings/ProtectingSensitiveData.md)
- Changelog: [doc/CHANGELOG.md](doc/CHANGELOG.md)
- Architecture decision records:
  [doc/adrs/index.md](doc/adrs/index.md)
- Quick links index: [index.md](index.md)

## Run locally

- `./mvnw spring-boot:run`
- Open `https://localhost:8443/hash` (accept the self-signed cert). You should
  see your data string, the cipher algorithm name, and the checksum value.
- If port 8443 is already in use, stop the process (for example,
  `lsof -i :8443` then `kill <pid>`) or override the port with
  `./mvnw spring-boot:run -Dspring-boot.run.arguments="--server.port=8444"`.
- Or auto-launch with: `python3 scripts/autolaunch.py` (starts the server,
  waits for port 8443, then opens the `/hash` page).

## Example SHA-256 Checksum Output from /hash Endpoint
  <img width="683" height="244" alt="Screenshot 2025-11-29 at 4 25 44 AM" src="https://github.com/user-attachments/assets/ee04e8c2-885d-4fd1-bce3-53e2cc809211" />


## Certificate Generation

- Requirements and commands:
  [doc/requirements/certificate-generation/requirements.md](doc/requirements/certificate-generation/requirements.md)
- Summary: use Java Keytool to generate a 2048-bit RSA self-signed certificate,
  export it to `server.cer`, and print the certificate details for verification.
- macOS/Linux example (replace with your own secure password):

  ```sh
  keytool -genkey -keyalg RSA -alias selfsigned \
    -keypass 'ExamplePass123!' \
    -keystore keystore.jks -storepass 'ExamplePass123!' \
    -validity 360 -keysize 2048

  keytool -exportcert -alias selfsigned \
    -keystore keystore.jks -storepass 'ExamplePass123!' \
    -file server.cer

  keytool -printcert -file server.cer
  ```

- Sample interactive output (placeholder values shown):

  ```
  Enter the distinguished name. Provide a single dot (.) to leave a sub-component empty or press ENTER to use the default value in braces.
  What is your first and last name?
    [Unknown]:  localhost
  What is the name of your organizational unit?
    [Unknown]:  ExampleOU
  What is the name of your organization?
    [Unknown]:  ExampleOrg
  What is the name of your City or Locality?
    [Unknown]:  ExampleCity
  What is the name of your State or Province?
    [Unknown]:  EX
  What is the two-letter country code for this unit?
    [Unknown]:  US
  Is CN=localhost, OU=ExampleOU, O=ExampleOrg, L=ExampleCity, ST=EX, C=US correct?
    [no]:  yes

  Generating 2048-bit RSA key pair and self-signed certificate (SHA384withRSA) with a validity of 360 days
          for: CN=localhost, OU=ExampleOU, O=ExampleOrg, L=ExampleCity, ST=EX, C=US
  ```

- Sample `keytool -printcert -file server.cer` output (placeholders):

  ```
  Owner: CN=localhost, OU=ExampleOU, O=ExampleOrg, L=ExampleCity, ST=EX, C=US
  Issuer: CN=localhost, OU=ExampleOU, O=ExampleOrg, L=ExampleCity, ST=EX, C=US
  Serial number: 1234567890abcdef
  Valid from: Mon Jan 01 00:00:00 EST 2025 until: Tue Dec 31 00:00:00 EST 2025
  Certificate fingerprints:
           SHA1: AA:BB:CC:DD:...:ZZ
           SHA256: 11:22:33:44:...:FF
  Signature algorithm name: SHA384withRSA
  Subject Public Key Algorithm: 2048-bit RSA key
  Version: 3
  ```
