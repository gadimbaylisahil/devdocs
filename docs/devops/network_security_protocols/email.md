# All about e-mail

References:

- [SMTP Relay](https://postmarkapp.com/blog/smtp-relay-services)
- [SMTP Details](https://postmarkapp.com/guides/everything-you-need-to-know-about-smtp#what-is-smtp)
- [SMTP Commands](https://www.samlogic.net/articles/smtp-commands-reference.htm)
- [SMTP Manual](https://smtpfieldmanual.com/)
- [DKIM](https://www.sparkpost.com/resources/email-explained/dkim-domainkeys-identified-mail/)

All protocols are implemented over TCP.

## Protocols

- SMPT(Simple Mail Transfer Protocol) **for sending**
- POP/IMAP **for receiving/retrieving**

### SMTP (Simple Mail Transfer Protocol)

- Is an e-mail protocol for **sending e-mails between addresses(push)** outside of your own domain.

- The only dedicated protocol for **sending(pushing)**.

#### SMPT Server

Handles sending e-mails between different addresses.

#### Ports

- 25: Unencrypted, do not use. Used in SMTP Relay, but skipped for client to server submissions.
- 465: Encrypted, Legacy, never standardized. Skip
- 587: Encrypted, preferred and standardized.
- 2525: Encrypted, use it when 587 is blocked.

### POP/IMAP

Protocols used to `receive/pull` the e-mails.

#### Ports

- POP - 110, 995(encrypted)
- IMAP 143, 993(encrypted)

#### POP (Post Office Protocol)

Not widely used, as you can only access e-mails from one machine only. Once you pull(download) e-mails, they are removed from the server. Good for offline access, as messages are pulled and saved locally.

#### IMAP (Internet Message Access Protocol)

IMAP uses cloud to store the e-mails and can be used to access/configure e-mails from different machines. Widely used due to convenience.

## Standards for more secure e-mail processing

- **SPF** allows senders to define which IP addresses are allowed to send e-mails for that particular domain.

- **DKIM** provides an encryption mechanism for transit.

- **DMARC** unifies the SPF and DKIM authentication methods into a framework, and allows domain owners to declare the way they would like email from that domain to be handled, in case it fails the authorization.

## SPF (Sender Policy Framework)

- Used to authenticate the sender of an e-mail

- SPF is a TXT record in DNS. It contains IP addresses that are allowed to send maail on behalf of the domain.

- ISPs can verify that a mail server is authorized to send e-mail for specific domain

## DKIM (DomainKeys Identified Mail)

- A way to verify that the sender is who it claim it is

- A way to encrypt headers and body

- Protects against spoofing and spam

- Uses public key cryptography

### How it works?

When e-mail is send, it is sent alongside a signature. This signature is verified by accessing the public key over the DNS records of the domain(TXT record).

Using the public key, message contents and headers are verified by the inbound mail server.

2 signatures, one for headers, one for the body.

Example:

```txt
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
 d=sparkpost.com; s=google;
 h=from:content-transfer-encoding:subject:message-id:date:to:mime-version;
 bh=ZkwViLQ8B7I9vFIen3+/FXErUuKv33PmCuZAwpemGco=;
 b=kF31DkXsbP5bMGzOwivNE4fmMKX5W2/Yq0YqXD4Og1fPT6ViqB35uLxLGGhHv2lqXBWwFhODP
```
###
