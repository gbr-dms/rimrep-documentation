# Data Sensitivity Classification

The following levels **do not** align with the Australian Government security classification system.

This document **does not** provide a framework for risk/impact assessment of unauthorised access or disclosure of data for the RIMReP-DMS project.

There are four levels of data sensitivity

- Open data
- Limited access data
- Sensitive data
- Highly sensitive data

## Open data

_Data that is publicly available or intended for public release without any restrictions. Examples could be general scientific research findings, aggregated statistics, or publicly accessible environmental data like air quality indices._

Open data is freely available to the public, and its access or disclosure poses <u>**minimal risk or impact**</u>.

## Limited access data

_Data that may be shared with a specific group or community but is not intended for unrestricted public dissemination. Examples include community-specific environmental data, research data shared among collaborators, or membership-based access to specific resources._

The unauthorized access or disclosure of limited access data may have a <u>**limited impact**</u>.

**The DMS is not currently able to handle limited access data**. It will be able to handle limited access data by the end of this phase of the project.

The following needs to happen before we can ingest limited access data:

- [ ] Authentication and authorisation system is in place and accepted by data providers
  - [ ] Including MFA
- [ ] Role-based access control
- [ ] Data sensitivity classification accepted by data providers
- [ ] Data sharing agreement framework accepted by data providers
- [ ] Auditing and monitoring to track user activities, data access, ...

## Sensitive data

_Data requires a higher level of protection due to potential risks associated with unauthorized access or disclosure. This may include personally identifiable information (PII), sensitive environmental data like detailed pollution reports, or specific locations of protected habitats._

The unauthorized access or disclosure of sensitive data can have <u>**significant adverse effects**</u>.

**The DMS is not currently able to handle sensitive data**. The DMS will not be able to handle sensitive data by the end of this phase of the project.

The following needs to happen before we can ingest sensitive data:

- [ ] All requirements for "limited access data"
- [ ] Data encryption (at rest and in transit)
- [ ] Data integrity and validation policy
- [ ] Data retention and disposal policy
- [ ] Incident response plan
- [ ] Security awareness and training for all staff handling sensitive data
- [ ] External security review of the DMS system
- [ ] Regular internal security assessments

## Highly sensitive data

_Data that requires the highest level of protection due to the severe consequences associated with unauthorized access or disclosure. Examples include highly sensitive personally identifiable information (PII), classified research, or data related to national security._

Unauthorized access or disclosure of highly sensitive data can lead to <u>**severe consequences**</u>.

**The DMS will never handle highly sensitive data**
