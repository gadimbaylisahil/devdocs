# All about e-mail

References:

- [SMTP Relay](https://postmarkapp.com/blog/smtp-relay-services)
- [SMTP Details](https://postmarkapp.com/guides/everything-you-need-to-know-about-smtp#what-is-smtp)
- [SMTP Commands](https://www.samlogic.net/articles/smtp-commands-reference.htm)
- [SMTP Manual](https://smtpfieldmanual.com/)

All protocols are over TCP.

## Protocols

- SMPT -> For sending
- POP/IMAP -> For receiving/retrieving

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

