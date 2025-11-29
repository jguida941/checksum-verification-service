## Protecting Sensitive Information

Credit card numbers. Social Security numbers. Passwords. Secret recipes for sugar
water. Eventually your application will need to handle sensitive information.
Whether you are writing the data to a file, storing it in a database, or sending
it across the network, your code needs to keep the information secure. In this
chapter, we cover techniques to keep data safe from prying eyes.

Before digging into those techniques, let’s first set the stage by reviewing the
threats that you need to block. Most people immediately think about the danger
of an eavesdropper harvesting confidential data as it passes over a network. To
be sure, that is a real threat, and the widespread adoption of the “cloud” means
that more applications than ever need to protect against this threat. We need a
way to send information across the network without worrying about eavesdroppers.

Another type of network-based threat is the attacker who isn’t interested in
the data you’re sending; she just wants to pretend to be you. These impersonators
want to be able to send messages that look like they came from your application.

For instance, let’s say you are building a web application to open a vault door
(it seems crazy, but someone somewhere is probably doing this). In order to open
the vault door, you send a specially crafted XML message that says: open sesame.
You might protect the message so that the attacker can’t determine how to say
“open sesame,” but if the attacker can watch you send the protected message,
then the attacker might be able to record that message and play it back later.
The attacker doesn’t know what the message is or how it works. She just knows
that if she sends the same message she watched you send, the vault opens. We
want to prevent this kind of impersonation.

Example (XML message with replay protection):

```xml
<VaultCommand id="12345" nonce="a8c9f3" timestamp="2024-05-01T12:00:00Z">
  <Action>open sesame</Action>
</VaultCommand>
```

To make this safe, sign or MAC the message (including the nonce and timestamp)
and have the server reject reused nonces or old timestamps. A simple pattern is
HMAC over the XML payload and metadata, storing nonces server-side to block
replay attempts.

In this example, `id` is a request identifier, `nonce` is a random value used
once to prevent replays, `timestamp` is when the message was created, and
`Action` holds the command. The client signs or MACs the full payload (including
nonce and timestamp), and the server checks the signature/MAC, ensures the
timestamp is fresh, and rejects any nonce it has already seen. That stops an
attacker from recording and reusing a prior open command.

Super short version:

- `id`: a label for this request so you can track it.
- `nonce`: a random one-time token; if it is seen twice, reject it.
- `timestamp`: when the request was made; if it is too old, reject it.
- `Action`: what you want done.
- The whole message gets signed or MACed, and the server only accepts it if the
  signature/MAC is valid, the timestamp is fresh, and the nonce is new.
- Practical note: you only need to remember nonces for a short window (for
  example, a few minutes) to block replays; storing them forever does not
  scale. If the message is already sent over TLS, basic network replay is
  mitigated at the transport layer; signing plus nonce/timestamp adds
  application-level replay protection.

## SQL Injection

SQL injection is a code injection technique that might destroy your database.
SQL injection is one of the most common web hacking techniques. SQL injection
is the placement of malicious code in SQL statements, via web page input.

### SQL in Web Pages

SQL injection usually occurs when you ask a user for input, like their
username/userid, and instead of a name/id, the user gives you an SQL statement
that you will unknowingly run on your database.

Look at the following example which creates a SELECT statement by adding a
variable (txtUserId) to a select string. The variable is fetched from user
input (getRequestString):

Example (Java, vulnerable):

```java
String txtUserId = request.getParameter("UserId");
String txtSQL = "SELECT * FROM Users WHERE UserId = " + txtUserId;

try (Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(txtSQL)) {
    // process results
}
```

Plain-English why it is bad: `txtUserId` is pasted straight into the SQL string,
so whatever the user types becomes part of the query the database executes.

The rest of this chapter describes the potential dangers of using user input in
SQL statements.

### SQL Injection Based on 1=1 is Always True

Look at the example above again. The original purpose of the code was to create
an SQL statement to select a user, with a given user id.

If there is nothing to prevent a user from entering "wrong" input, the user can
enter some "smart" input like this:

UserId:

Then, the SQL statement will look like this:

```sql
SELECT * FROM Users WHERE UserId = 105 OR 1=1;
```

Why it returns everything: `OR 1=1` is always true, so the WHERE clause matches
every row and the filter is effectively removed.

The SQL above is valid and will return ALL rows from the "Users" table, since
OR 1=1 is always TRUE.

Does the example above look dangerous? What if the "Users" table contains names
and passwords?

The SQL statement above is much the same as this:

```sql
SELECT UserId, Name, Password FROM Users WHERE UserId = 105 or 1=1;
```

Why it leaks data: the same always-true condition means the query ignores the
intended user filter and returns every user and password.

A hacker might get access to all the user names and passwords in a database, by
simply inserting 105 OR 1=1 into the input field.

### SQL Injection Based on ""="" is Always True

Here is an example of a user login on a web site:

Username:

Password:

Example (Java, vulnerable):

```java
String uName = request.getParameter("username");
String uPass = request.getParameter("userpassword");

String sql = "SELECT * FROM Users WHERE Name =\"" + uName
    + "\" AND Pass =\"" + uPass + "\"";

try (Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery(sql)) {
    // process results
}
```

Plain-English why it is bad: `uName` and `uPass` are dropped directly into the
query text, so an attacker can add SQL logic instead of just providing
credentials.

Result

```sql
SELECT * FROM Users WHERE Name ="John Doe" AND Pass ="myPass"
```

What the normal query looks like: with typical input it selects one user; with
malicious input the same template can be turned into an always-true login.

A hacker might get access to user names and passwords in a database by simply
inserting " OR ""=" into the user name or password text box:

User Name:

Password:

The code at the server will create a valid SQL statement like this:

Result

```sql
SELECT * FROM Users WHERE Name ="" or ""="" AND Pass ="" or ""=""
```

Why it matches everything: `""=""` is always true, so the WHERE clause evaluates
to true for every row.

The SQL above is valid and will return all rows from the "Users" table, since
OR ""="" is always TRUE.

### SQL Injection Based on Batched SQL Statements

Most databases support batched SQL statement.

A batch of SQL statements is a group of two or more SQL statements, separated
by semicolons.

Whether multiple statements are executed depends on the database and driver
configuration; some drivers disable multi-statement execution or require an
explicit setting to allow it.

The SQL statement below will return all rows from the "Users" table, then
delete the "Suppliers" table.

Example

```sql
SELECT * FROM Users; DROP TABLE Suppliers
```

What is happening: if the database/driver allows multiple statements, this one
query both reads all users and then drops the table immediately after.

Look at the following example:

Example (Java, vulnerable):

```java
String txtUserId = request.getParameter("UserId");
String txtSQL = "SELECT * FROM Users WHERE UserId = " + txtUserId
    + "; DROP TABLE Suppliers;";

try (Statement stmt = conn.createStatement()) {
    stmt.execute(txtSQL);
}
```

Why it is dangerous: because `txtUserId` is concatenated, an attacker can append
`; DROP TABLE ...` and the driver may execute both statements.

And the following input:

User id:

The valid SQL statement would look like this:

Result

```sql
SELECT * FROM Users WHERE UserId = 105; DROP TABLE Suppliers;
```

Outcome: the injected user id produces a select followed by the destructive drop
of the Suppliers table.

### Use SQL Parameters for Protection

To protect a web site from SQL injection, you can use SQL parameters.

SQL parameters are values that are added to an SQL query at execution time, in
a controlled manner.

Parameters only protect the parts of the query you bind; dynamic identifiers
like column names or ORDER BY fragments still need whitelisting. You also need
least-privilege database accounts and input validation to match business rules.

Example (Java, using parameters):

```java
String txtUserId = request.getParameter("UserId");
String txtSQL = "SELECT * FROM Users WHERE UserId = ?";

try (PreparedStatement ps = conn.prepareStatement(txtSQL)) {
    ps.setString(1, txtUserId);
    try (ResultSet rs = ps.executeQuery()) {
        // process results
    }
}
```

Why this is safer: the `?` placeholder is bound to `txtUserId`, so the database
treats it as a value even if it contains SQL-looking text; it cannot change the
query structure.

Note that parameters are represented in the SQL statement by a ? marker.

The SQL engine checks each parameter to ensure that it is correct for its
column and are treated literally, and not as part of the SQL to be executed.

Another Example

```java
String txtNam = request.getParameter("CustomerName");
String txtAdd = request.getParameter("Address");
String txtCit = request.getParameter("City");

String txtSQL = "INSERT INTO Customers (CustomerName, Address, City)"
    + " VALUES (?, ?, ?)";

try (PreparedStatement ps = conn.prepareStatement(txtSQL)) {
    ps.setString(1, txtNam);
    ps.setString(2, txtAdd);
    ps.setString(3, txtCit);
    ps.executeUpdate();
}

```

Why this is safer: `txtNam`, `txtAdd`, and `txtCit` are user inputs, but the SQL
uses ? placeholders and `PreparedStatement` binds each value in order, so the
database treats them purely as data, not as SQL to run.

Examples

The following examples shows how to build parameterized queries in Java.

SELECT STATEMENT:

```java
String txtUserId = request.getParameter("UserId");
String sql = "SELECT * FROM Customers WHERE CustomerId = ?";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, txtUserId);
    try (ResultSet rs = ps.executeQuery()) {
        // process results
    }
}
```

Why this is safer: binding the customer id with a `?` parameter keeps the input
as data only; it cannot alter the SELECT or add extra SQL.

INSERT INTO STATEMENT:

```java
String txtNam = request.getParameter("CustomerName");
String txtAdd = request.getParameter("Address");
String txtCit = request.getParameter("City");

String txtSQL = "INSERT INTO Customers (CustomerName, Address, City)"
    + " Values(?, ?, ?)";

try (PreparedStatement ps = conn.prepareStatement(txtSQL)) {
    ps.setString(1, txtNam);
    ps.setString(2, txtAdd);
    ps.setString(3, txtCit);
    ps.executeUpdate();
}
```

Why this is safer: each user input is bound to a `?` placeholder, so the INSERT
receives literal values; user input cannot inject SQL into the statement.


## Java Cryptography Architecture (JCA)

The JCA is the foundational framework for accessing and developing cryptographic
functionality within the Java platform. It provides:

- A Provider-Based Architecture: JCA separates the cryptographic API from its
  implementation. This allows developers to use different cryptographic
  providers (implementations of cryptographic algorithms) without changing their
  application code.
- APIs for Core Cryptographic Concepts: JCA includes APIs for:
  - Digital Signatures: Ensuring the authenticity and integrity of data.
  - Message Digests (Hashes): Generating fixed-size representations of data for
    integrity checks.
  - Certificates and Certificate Validation: Managing and verifying digital
    certificates.
  - Key Management: Generating, storing, and managing cryptographic keys.
  - Secure Random Number Generation: Providing cryptographically strong random
    numbers.

## Java Cryptography Extension (JCE)

The JCE extends the JCA by providing support for additional cryptographic
algorithms and operations, particularly those related to encryption and key
exchange. JCE includes APIs and implementations for:

- Encryption and Decryption: Supporting various ciphers, including symmetric
  (e.g., AES) and asymmetric (e.g., RSA) algorithms, as well as block and stream
  ciphers.
- Key Generation and Agreement: Facilitating the generation of cryptographic
  keys and the secure establishment of shared secret keys between parties.
- Message Authentication Codes (MACs): Providing mechanisms to verify the
  integrity and authenticity of messages.

In essence, JCA provides the core framework for cryptographic services, while
JCE expands upon this framework to include more advanced and widely used
cryptographic functionalities, particularly in the realm of encryption and key
management. Together, they offer a robust and flexible platform-independent
solution for integrating security into Java applications.


## Securing Data in Transit

The standard way to protect against the network-based threats mentioned earlier
is to use the cryptographic protocol Secure Sockets Layer (SSL) or its modern
incarnation, Transport Layer Security (TLS). The original SSL specification was
developed and released by Netscape in 1995. The original protocol standard has
been revised several times to adjust for weaknesses in the specification and
new security techniques. In 1999, the standard became known as TLS, and the
current widely deployed versions are TLS 1.2 and TLS 1.3 (TLS 1.3 is
recommended; TLS 1.2 is acceptable when configured securely). Despite the name
change, most people continue to use SSL to refer to both SSL and TLS.

NOTE

The HTTP Strict Transport Security (HSTS) standard is a well-supported HTTPS
response Header that will force a browser to always use HTTPS for a certain
length of time! Simply deliver the following as an HTTPS response header to
ensure that supported browsers can only make HTTPS connections to your site for
max-age number of seconds!

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

The Chromium project will even allow you to preload HSTS headers so that users
of Chrome, Aviator, and other Chromium-based projects will automatically get
the benefit of Strict Transport Security on the first hit to your site.2 Major
browsers maintain an HSTS preload list you can submit your domain to, so HTTPS
is enforced from the very first request.

The backbone of these protocols is provided by a public key infrastructure that
allows you to trust servers that you’ve never worked with before. This is
accomplished by a system of certificate authorities (CAs), such as DigiCert,
GlobalSign, and Let’s Encrypt, which sign cryptographic certificates that are
placed on servers. When a TLS client connects to a server, the server sends the
certificate to the client. If the client trusts the CA that signed the
certificate, then the client may trust the server after verifying that the
authority actually did sign the certificate. This verification step is made
possible by the public key cryptography on which TLS is based. Once this trust
is established, a secure communication channel is created.

Certificate chains are used to provide better scalability and control of
certificates. A certificate chain uses two types of certificate authorities:
root CAs that clients trust and intermediate CAs that have certificates signed
by the root CA. If the certificate that is used by a TLS server is signed by an
intermediate CA, then the server will provide both certificates to the client:
the server certificate and the intermediate CA certificate.

The client will then verify the chain. First it verifies that the server
certificate is signed by the intermediate CA; then it verifies that the
intermediate CA’s certificate is signed by a root CA that the client trusts. If
the chain is unbroken, then the client will trust the server. A chain can
consist of certificates from several intermediate CAs. In that case, the server
will provide the entire chain, and the client will need to verify the entire
chain.

The client maintains a list of root certificates from CAs that it trusts.
Practically speaking, these root certificates are distributed with browsers,
SDKs, and other applications. Java comes with a set of trusted certificates
stored in the file $JAVA_HOME/lib/security/cacerts. We look at how to read that
file later in the chapter.

As you can see, TLS is a complicated protocol. One TLS 1.2 specification, RFC
5246, is 104 pages long and refers to other RFCs. Most of the time you will not
need to worry about the details of TLS. If you are writing code that runs
behind a web server, then you will rely on the web server to handle all of the
connection details. In that case, most of this section won’t be directly
relevant. However, you will still need to configure the certificates used by
the web server and that topic is covered by the “Certificate and Key
Management” section.

The same considerations apply if you are writing code that runs behind a web
client that already handles the connection to a web server. You’ll need to
configure the certificates but not worry about the low-level connection
details.

However, be careful about relying too much on your web server or client front
end. It is surprising how many of these don’t provide the security checks that
you expect.3 If you depend on a front end, test it with all sorts of
misconfigured certificates to see how it responds. Use expired certificates and
certificates with incorrect hostnames. Let’s hope that your front end
terminates the connection when confronted with bad certificates, but research
has shown that many implementations carry on anyway and are therefore
vulnerable to a variety of attacks that TLS/SSL should prevent if used
correctly.

With that out of the way, let’s move on to TLS and SSL. Java makes it very easy
to use these protocols. The Java Secure Sockets Extension (JSSE) consists of
several packages that make dealing with SSL and TLS fairly straightforward. How
straightforward? Simply import the javax.net.ssl.* package, define the host and
port, and create a socket:
```Java
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.SSLSocket;
import java.net.Socket;

SSLSocketFactory sf = (SSLSocketFactory) SSLSocketFactory.getDefault();
try (SSLSocket socket = (SSLSocket) sf.createSocket(host, port)) {
    // use socket.getInputStream() / getOutputStream()
}
```

Plain-English: this makes a client TLS socket using the default SSL settings,
so the library does the handshake, encrypts traffic, and checks the server’s
certificate for you.

You now have a secure communications channel that can be used like any other
Java socket connected to a server.

Creating a secure server socket is equally straightforward:

```Java
import javax.net.ssl.SSLServerSocketFactory;
import javax.net.ssl.SSLServerSocket;
import java.net.ServerSocket;

SSLServerSocketFactory ssf =
    (SSLServerSocketFactory) SSLServerSocketFactory.getDefault();
try (SSLServerSocket ss = (SSLServerSocket) ssf.createServerSocket(port)) {
    // accept TLS clients here
}
```

Plain-English: this makes a server TLS socket that will negotiate TLS for
incoming connections (handshake, encryption, cert exchange) using the default
SSL settings.

Using these SSL/TLS sockets means the JSSE layer handles the TLS handshake,
negotiates encryption, and validates certificates so data in transit stays
confidential and tamper-resistant. It also lets clients verify server identity
via certificates, which is why it is safer than a plain socket. You still need
to configure the right certificates and trust settings, but the heavy lifting
for secure transport is built in.

Note: hostname verification is automatic in higher-level HTTP clients (like
HttpsURLConnection). With raw sockets, set
`SSLParameters#setEndpointIdentificationAlgorithm("HTTPS")` or do your own
hostname check so you validate both the certificate chain and the host name.

In both cases, host is a string that contains the hostname and port is an
integer that represents the port over which the connection should be
established. For instance, the standard port for HTTPS is 443.

While this is easy enough, it isn’t the whole story. In security, there are
always details to keep in mind, and these details, if misconfigured, can reduce
the security of an otherwise rock-solid security system.

Plain-English takeaway: use these SSL/TLS sockets so JSSE encrypts traffic and
verifies certificates for you; your job is to supply the right certificates and
trust settings.

Plain-English trust flow (why the TLS/cert checks matter):

- Your device already has a trusted CA list (OS/browser/Java truststore).
- A site gets a CA-signed cert (“ID card”) proving it controls the domain.
- On connect, the server sends its cert chain; your client checks trust, expiry,
  and that the name matches the host.
- The server also proves it owns the private key during the handshake.
- You accept only if: the chain is trusted, the name matches, the date is OK,
  and the server proves key possession.
- Attackers fail unless: a CA is compromised, your truststore is tampered with,
  or you ignore warnings (for example, “trust all” trust managers or permissive
  hostname verifiers).

## Protocol Versions

Because SSL and TLS have a long history, you want to make sure that you are
using only the latest secure protocols. You can control what protocols the
socket will use. The following fragment enables just TLS versions 1.2 and 1.3
on the socket:
```Java
String[] p = { "TLSv1.2", "TLSv1.3" };
socket.setEnabledProtocols(p);
```
TLS 1.2 and 1.3 are the current state of the art at the time of this writing
and are the ones you should use if you can. However, not all systems support
these two protocols. You will likely encounter systems that support version 1.0
of TLS and various versions of SSL. Try to stay away from SSL, and instead use
one of the TLS versions—preferably the most modern version that is supported by
your systems. One TLS 1.2 specification is RFC 5246; TLS 1.3 is RFC 8446.

Recent JDKs already disable SSLv3 and often TLS 1.0/1.1 by default; use
setEnabledProtocols to tighten further or to keep compatibility with legacy
peers, not to re-enable old protocols in new code.

- SSL 2.0 and SSL 3.0: obsolete and insecure; must be disabled.
- TLS 1.0 and TLS 1.1: deprecated and vulnerable to several attacks; should be
  disabled.
- TLS 1.2: widely used and secure when configured with strong ciphers and good
  settings.
- TLS 1.3: current recommended protocol; simpler design and removes many legacy
  weaknesses.

Examples of why the JSSE code matters: the client example shows how to wrap an
outbound TCP connection in TLS so secrets are encrypted and the server’s
certificate is checked; the server example shows how to accept inbound TLS
connections so clients can trust your service and data in transit is protected.

## Cipher Suites
The security of the socket depends on two things: keeping the keys secure and
using appropriately strong encryption. We’ll cover key security later, so in
this section we discuss encryption algorithms.

TLS uses different algorithms at various stages of creating and managing a
secure socket. These combinations of algorithms are referred to as cipher
suites and they are legion. Here’s a sample:

```text
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_DHE_RSA_WITH_AES_256_CBC_SHA
TLS_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
```

Note: CBC examples above are for illustration of naming; prefer AEAD suites
(AES-GCM, ChaCha20-Poly1305) for new configurations.

The first part identifies these as TLS, as opposed to SSL, cipher suites. This
is followed by the key exchange and authentication algorithms. On the other
side of the “WITH” is the specification for the bulk encryption cipher, its key
size and mode, and the message authentication algorithm. This pattern can vary,
but it’s enough to give you an idea of how to read the cipher suite names.

It’s not easy to give a quick recommendation as to what suite you should use.
The JSSE defaults are good, so try to avoid changing them if possible. If you
must customize your cipher suites, remember that the world of cryptography
changes quickly, and you’ll need to stay abreast of new attacks and algorithms
and appropriately adjust your supported suites. General rules are:4

- Don’t use suites that list ANON for authentication. They don’t provide
  authentication.
- Don’t use suites that contain NULL.
- Avoid use of suites that contain EXPORT.
- Stick to bulk ciphers with key sizes of 128 bits or larger (note that 3DES
  provides no more than 112 bits of security).
- Try to avoid suites using RC4, DES, and 3DES.
- Prefer ECDHE and DHE for key agreement. While they are slower (and DHE is
  slower than ECDHE), they provide stronger protection even if the private
  keys are later compromised, a property known as forward secrecy.

Example breakdown of keywords:

- `TLS`: protocol family (avoid SSL).
- `ECDHE` / `DHE`: ephemeral Diffie-Hellman key exchange (forward secrecy);
  `RSA`: RSA key exchange; `ANON`: no authentication (avoid).
- `RSA` / `ECDSA`: authentication algorithms using RSA or ECDSA keys.
- `AES_256_GCM` / `AES_128_CBC` / `CHACHA20_POLY1305`: bulk encryption
  algorithm and mode; prefer AEAD modes like GCM or ChaCha20-Poly1305.
- `SHA256` / `SHA384`: hash/MAC used for integrity.
- `NULL`: no encryption (avoid); `EXPORT`: weak, legacy export ciphers (avoid).

So which suites meet these guidelines? You might use one of these two:
```text
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
```

## Forward Secrecy

Forward secrecy protects your past communications even if the server’s private
keys are compromised. Without forward secrecy, an attacker who gains access to
a server’s private key can decrypt communications by applying the server’s
private key to stored traffic that the attacker recorded in the past. With
forward secrecy, the attacker would be able to use the private key to
impersonate the server, but the attacker would not be able to decrypt recorded
communication traffic.

Stealing private keys is not as difficult as you might think. Trusted system
administrators typically have access to the keys, a warrant can compel
administrators to hand over the keys, and security vulnerabilities can leave
the keys exposed. For instance, the OpenSSL Heartbleed vulnerability,
publicized in April 2014, enabled remote attackers to steal a server’s private
keys by sending messages that abused the heartbeat extension.

Systems that had implemented forward secrecy were not exposed to as much risk
by the Heartbleed vulnerability (not to diminish the severity of Heartbleed)
because their past communications were still secure. Google enabled forward
secrecy in many of its sites in 2011. Twitter did so in 2013. We can’t stress it
enough: Protect your communications by selecting cipher suites that support
forward secrecy (ECDHE or DHE).

TLS 1.3 always uses ephemeral Diffie-Hellman, so forward secrecy is built in.
For TLS 1.2, you need to pick suites that use ECDHE or DHE to get forward
secrecy.

Modern browsers and servers widely support ECDHE; the main compatibility issues
are with very old clients. DHE also provides forward secrecy but is slower and
sometimes disabled. Test that both ends share at least one ECDHE/DHE suite; if
they do not, the connection fails with a handshake_failure before any data is
sent.

The code needed to specify the cipher suites is simple enough:

```Java
String[] ciphers = {
    "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
    "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
    "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
    "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256"
};
socket.setEnabledCipherSuites(ciphers);
```

Now your socket will use only those AEAD ECDHE suites (forward secrecy plus
modern encryption). If you control both endpoints, ensure these suites are
supported on both sides. If you control only one side, research and test for
compatibility with your peers; very old clients might not support these suites.
Plain-English: pick ECDHE-based AEAD suites for TLS 1.2; TLS 1.3 already has
forward secrecy built in.


## Certificate Verification

Recall that impersonation is one of the threats that you want to block. Two key
techniques to block impersonation are verifying that the server’s hostname
matches the hostname noted in the certificate, and verifying that the
certificate has not expired. By default, secure sockets in Oracle’s JSSE will
verify the expiration date but not the hostname. This is because hostname
verification varies depending on the protocol built on top of TLS while
expiration date checking is the same across protocols. We’ll look at a subtlety
in expiration date checking in a moment, after we first look at hostname
verification.

While the low-level secure sockets don’t verify the hostname, the HTTPS support
in JSSE does. To accomplish this, the classes that implement HTTPS include a
HostnameVerifier, which verifies that the hostname in the certificate matches
the name of the host to which you are connecting. If they do not match, the
connection will terminate.

For instance, the following code will establish an HTTPS connection to
your.url.com. If the certificate at your.url.com does not match the your.url.com
hostname, this connection will terminate before any data is sent.
```Java
String hostURL = "https://your.url.com/resource";
URL url = new URL(hostURL);
HttpsURLConnection ctn;
ctn = (HttpsURLConnection) url.openConnection();
```

The hostname check is complicated and a bit fussy. As a result, it will
sometimes fail in cases where you would rather it succeed. To deal with this
situation, you can implement your own HostnameVerifier that will be called in
the event of a hostname mismatch. The following code shows the skeleton of a
custom hostname verifier that always returns false.
```Java
import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLSession;
public static class CustomHostnameVerifier implements HostnameVerifier {

  @Override
  public boolean verify(String hostname, SSLSession session) {
    boolean verified = false;
    // TODO: real hostname verification logic here
    return verified;
  }
}
```

Add your own logic to verify the host, but understand that hostname verification
is more complicated than it sounds.5

A single line associates your hostname verifier with the HTTPS connection. In
the code that follows, we add the custom verifier to an HTTPS connection.

```Java
public class Example {

  public void connect() throws Exception {
    String hostURL = "https://your.url.com";
    URL url = new URL(hostURL);
    HttpsURLConnection ctn =
            (HttpsURLConnection) url.openConnection();

    ctn.setHostnameVerifier(new CustomHostnameVerifier());

    // trigger the connection
    ctn.connect();
  }
}
```

A common case for custom hostname verifiers is to accommodate development
environments. Certificates on development machines may not match exactly, and
rather than spend the time fixing the certificates, many developers will simply
use their own hostname verifiers as workarounds. Just make sure that your
development code doesn’t run in production.6

If you need HTTPS-style hostname validation in a low-level secure socket
connection, as opposed to an HTTPS connection, you can use the SSLParameters
class, as shown here:

```Java
SSLParameters sslParams = new SSLParameters();
sslParams.setEndpointIdentificationAlgorithm("HTTPS");
socket.setSSLParameters(sslParams);
```

The socket in the preceding code will now verify the server’s hostname using
the rules for verifying hostnames in an HTTPS connection.

Let’s now take a quick look at the subtleties in checking a certificate’s
expiration date, as mentioned earlier. As you would expect, if a certificate is
part of a certificate chain and the certificate has expired, JSSE will throw an
exception before establishing the connection and transmitting data. However, if
the client explicitly trusts the server (for example, a custom truststore or
trust manager that directly accepts the server cert), the socket will connect
and encrypt traffic even if the certificate has expired. In this case,
“explicitly” means that the server’s certificate has been added directly to the
client’s truststore and the certificate’s validity is not verified using a
certificate chain.7 In this case, you need to check a certificate’s expiration
date yourself using the checkValidity() method of the X509Certificate class. We
show an example of that in the next section on trust managers. To be clear
though, this is an edge case and is not needed when you are relying on
validation via a CA’s signature.

## Trust Managers
