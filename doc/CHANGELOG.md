# Changelog

All notable changes to this project are documented here.

## 2025-11-29

- Added `scripts/autolaunch.py` to start the app, wait for port 8443, and open
  the `/hash` page automatically; documented helper in README.
- Documented how to handle port 8443 conflicts and clarified `python3`
  autolaunch usage in the README.
- Wrapped and refreshed the submission template with clear placeholders and a
  repository-friendly screenshot link location.
- Moved checksum requirements to `doc/requirements/checkSum-Verification/` and
  added certificate-generation requirements under
  `doc/requirements/certificate-generation/`.
- Updated `/hash` endpoint response to clearly show data, cipher algorithm name,
  and checksum value; documented run/endpoint usage in README.
- Recorded ADR 0001 capturing the decision to use SHA-256 for checksum
  generation.
- Added ADR folder with template and guidance for recording decisions.
- Reorganized docs into subfolders (requirements, templates, readings) for
  clarity and updated links.
- Added ADR index for quick navigation of decision records.
- Added a project README and changelog reference for easy navigation.
- Initial repository setup, docs, and Spring Boot checksum service scaffold.
