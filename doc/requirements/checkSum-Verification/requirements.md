# Requirements for Module Five Coding Assignment: Checksum Verification

A business you are working with has a public key that it hopes to distribute to
clients. The business' clients want to download the public key from a website
and verify the key with a checksum. To accomplish this, you will code a simple
string checksum verification program. You will also document your process by
completing the Module Five Coding Assignment Checksum Verification Template
linked in the What to Submit section. Review the module's Resources section to
help you with this assignment.

Specifically, you must address the following rubric criteria:

- Algorithm Cipher: Recommend an appropriate encryption algorithm cipher that
  avoids collisions.
  - Review the scenario and the Java Security Standard Algorithm Names resource
    linked in the Supporting Materials section. The Java Security Standard
    Algorithm Names is a standard list of algorithm ciphers provided by Oracle.
  - Document your recommendation in the template provided.
  - Status: [x] Completed 2025-11-29 - documented in
    `doc/templates/ChecksumVerificationTemplate.md`.

- Justification: Justify your reasoning for the recommended algorithm cipher
  that avoids collisions.
  - Provide a brief, high-level overview of the encryption algorithm software.
    Consider what avoiding collisions means. Why is avoiding collisions
    important?
  - Document your reasoning in the template provided.
  - Status: [x] Completed 2025-11-29 - documented in
    `doc/templates/ChecksumVerificationTemplate.md`.

- Generate Checksum: Refactor the code to encrypt a text string and generate a
  checksum verification.
  - Download the Module Five Coding Assignment Checksum Verification Code Base,
    linked in the Supporting Materials section, and upload it to Eclipse as a
    new project. Refactor the code to add your first name and last name as a
    unique data string. You will submit your refactored code for your
    instructor to review. Then generate the checksum by following these steps:
    - Create an object of MessageDigest class using the
      java.security.MessageDigest library.
    - Initialize the object with your selection for an appropriate algorithm
      cipher.
    - Use the digest() method of the class to generate a hash value of byte
      type from the unique data string that includes your first name and last
      name.
    - Convert the hash value to hex using the bytesToHex function.
    - Create a RESTFul route using the @RequestMapping method to generate and
      return the required information, including the hash value, to the secure
      web browser.
  - Status: [x] Completed 2025-11-29 - implemented in
    `src/main/java/com/snhu/sslserver/ServerApplication.java` with the `/hash`
    endpoint returning the data string, algorithm name (SHA-256), and checksum.

- Verification: Demonstrate that a hash value has been created for the unique
  text string that includes your first name and last name by executing the Java
  code.
  - Then use your web browser to connect to the RESTful API server. This action
    should show your first name and last name as the unique data string in the
    browser, the name of the algorithm cipher you used, and the checksum hash
    value.
  - Capture a screenshot of the secure web browser with your unique
    information and add it to the template provided. An example of the expected
    output is shown below.
  - Status: [x] Completed 2025-11-29 - screenshot added to
    `doc/templates/ChecksumVerificationTemplate.md` and referenced in README.
