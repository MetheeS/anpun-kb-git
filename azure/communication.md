# azure — communication

## [acs-email-contenttype-and-verified-sender]
created: 2026-06-22
tags: acs, communication-services, email, sdk, sender-domain
symptom/context: Azure Communication Services (ACS) email send fails or sends a
  malformed message: (1) a DomainNotLinked error, or (2) the body does not render
  / is set via the wrong SDK field.
root-cause: (1) The sender From address MUST belong to a domain that is LINKED to
  the ACS resource; a domain that is verified but not linked to that resource
  yields DomainNotLinked. (2) The GA azure-communication-email SDK sets the body
  via content.{subject, plainText, html}; the older PREVIEW SDK's attachmentType
  field is not how you set the MIME body — code carried over from the preview SDK
  sends an empty/incorrect body.
fix: Use a sender on a domain LINKED to the ACS resource (check the resource's
  linked domains, not just domain verification). Build the message with the GA
  SDK content fields (plainText/html), not the preview attachmentType.
