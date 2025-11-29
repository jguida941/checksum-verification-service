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
