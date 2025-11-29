CS 305 Module Five Coding Assignment Checksum Verification Template

Instructions
Using the instructions from the Module Five Coding Assignment Checksum
Verification Guidelines and Rubric, replace the bracketed text with the
relevant information in your own words.

1. Algorithm Cipher

I recommend using SHA-256 as the MessageDigest algorithm.

2. Justification

SHA-256 is a modern SHA-2 hash algorithm provided by Java
(MessageDigest.getInstance("SHA-256")). It is designed to be
collision-resistant, and there are no practical collision attacks known
against it. Legacy algorithms like MD5 and SHA-1 from the same Java standard
list are known to be vulnerable to collisions, so they are not appropriate for
verifying a published public key by checksum.

I also considered SHA-512, which provides an even larger security margin, but
it produces a longer hash value (512 bits vs. 256 bits) that is unnecessary for
this use case. SHA-256 is widely supported and typically faster to compute than
SHA-512, making it a good balance of security and performance for this
application.

3. Generate Checksum

You’ll submit your refactored code to your instructor. Your instructor will
review it and this document.

4. Verification

Insert a screenshot below of the web browser with your unique information.

[Insert screenshot.]
