## Table of contents

- [Metadata](#metadata)
  - [Security links](#security-links)
- [Overview](#overview)
  - [Actors](#actors)
  - [Actions](#actions)
  - [Background](#background)
  - [Goals](#goals)
  - [Non-goals](#non-goals)
- [Self-assessment use](#self-assessment-use)
- [Security functions and features](#security-functions-and-features)
- [Project compliance](#project-compliance)
- [Secure development practices](#secure-development-practices)
- [Security issue resolution](#security-issue-resolution)
- [Appendix](#appendix)

## Metadata

|                   |                                                                  |
| ----------------- | ---------------------------------------------------------------- |
| Software          | https://github.com/etcd-io/etcd                                  |
| Security Provider | No                                                               |
| Languages         | Golang                                                           |
| SBOM              | https://github.com/etcd-io/etcd/blob/main/bill-of-materials.json |
|                   |                                                                  |

### Security links

| Doc                      | url                                                                            |
| ------------------------ | ------------------------------------------------------------------------------ |
| Security Audit           | https://github.com/etcd-io/etcd/blob/main/security/SECURITY_AUDIT.pdf          |
| Security Release Process | https://github.com/etcd-io/etcd/blob/main/security/security-release-process.md |
| Security Policy          | https://github.com/etcd-io/etcd/blob/main/security/README.md                   |

## Overview

Etcd, a distributed key-value store, serves as the backbone for reliable and fault-tolerant data storage in distributed systems. It is characterized by its simplicity, security, efficiency and reliability and is used in production by many companies.

### Background

- **Origin and Development:**

  - Origins: etcd was initially developed by CoreOS, a company focused on container technologies and Linux distribution. Originally, etcd was developed to manage a cluster of CoreOS Container Linux. In 2014, Google launched the Kubernetes project and used etcd for their configuration store.
  - Release Date: The first version of etcd was released in June 2013.

- **Purpose and Design:**

  - Distributed Systems: The primary purpose of etcd is to provide a distributed and reliable key-value store for managing configuration data, state, and metadata in distributed systems.
  - Consistency and Fault Tolerance: etcd is designed to maintain high consistency and fault tolerance through the use of the Raft consensus algorithm.

- **Key Features:**

  - Raft Consensus Algorithm: etcd employs the Raft algorithm to achieve consensus among distributed nodes. Raft helps ensure that all nodes in the etcd cluster agree on the state of the data, even in the presence of failures.
  - Watch Mechanism: etcd supports a watch mechanism, allowing applications to be notified of changes to specific keys. This is useful for building reactive systems that respond to changes in configuration or state.
  - HTTP/JSON API: It provides a simple HTTP/JSON-based API for interacting with the key-value store, making it accessible and easy to integrate into various applications.
  - Security Features: etcd includes security features such as Transport Layer Security (TLS) for encrypting communication between nodes and authentication/authorization mechanisms for controlling access to the key-value store.

- **Adoption and Community:**

  - Cloud Native Computing Foundation (CNCF): etcd became a project hosted by CNCF, which is known for fostering the development of open-source cloud-native technologies.
  - Community Involvement: The project has gained a strong community of contributors and users. Open-source collaboration has played a significant role in its development and improvement.

- **Use Cases:**

  - Kubernetes: etcd is a critical component in Kubernetes, the popular container orchestration platform. It is used to store configuration data, cluster state, and other critical information.
  - Distributed Systems: Beyond Kubernetes, etcd is employed in various distributed systems where a reliable and distributed key-value store is needed.

- **Open Source and License:**

  - Licensing: etcd is released under the Apache 2.0 open-source license, allowing users to use, modify, and distribute the software freely.
  - GitHub Repository: The source code and development discussions are hosted on
  - GitHub, making it accessible to the open-source community.

- **Evolution and Updates:**

  - Regular Releases: The etcd project has a history of regular releases, introducing new features, improvements, and bug fixes to enhance its capabilities.
  - Compatibility: New releases often consider compatibility with existing deployments to facilitate smooth upgrades.

- **Ecosystem Integration:**

  - Tooling: etcd has various operational tools for monitoring, backup, and restore operations. Integration with other tools in the cloud native ecosystems is common.

### Actors

**etcd Server**

Etcd server is responsible for managing the distributed key-value store. These server nodes communicate with each other using Raft consensus algorithm to maintain consistency and resilience.

Etcd server nodes are isolated from other actors as they are typically deployed on separate machines. Any failure on part of the server nodes will not cause significant errors or security issues.

**etcd Balancer**

The Etcd Balancer assumes the crucial responsibility of efficiently distributing client requests to enhance the performance of Etcd services, preventing any single server node from being overwhelmed. In accordance with the latest standard (clientv3-grpc1.23), the Etcd Balancer establishes and manages separate connections for each server node. Following a default round-robin policy, the Etcd Balancer directs incoming requests to each server node in sequential order. When a server node experience a failure, the Etcd Balancer excludes it from consideration until it is restored to the network. This strategic approach contributes to the robust and reliable operation of Etcd services.

**etcd Clients**

Etcd clients are responsible for interacting with the etcd servers, making gRPC requests to the distributed key-value store. Etcd clients could query, update, delete the key-value pair in the etcd server, they can also watch real-time changes on the keys and detect client client liveness.

Etcd clients are isolated from other actors since they are external entities that communicate with the etcd cluster.

**etcd Storage Backend**

The etcd storage backend stores the key-value data persistently. It employed various technologies like multi-version concurrency control to ensure consistency and concurrent transactions. Update operations will not replace the data but create a new key-value pair with increment revision number. Therefore, all past versions of keys are still avaliable for accessing.

### Actions

1. **Key-Value Store Operations:**

   - **Performed by:** etcd Server Nodes, etcd Clients, etcd Balancer
   - **Description:** etcd client could do various key-value store operations, including reads, writes, deletions, watching, compacting and more. These operations are distributed across Etcd server nodes, ensuring both consistency and concurrency through the robust Raft consensus protocol and the Etcd storage backend. To optimize performance, all requests are systematically dispatched by the Etcd Balancer, evenly distributing them across all Etcd server nodes. This approach contributes to the effective and balanced execution of operations within the Etcd environment.

2. **Raft Consensus Protocol:**

   - **Performed by:** etcd Server Nodes
   - **Description:** The Etcd server employs the Raft consensus protocol for distributed coordination among multiple server nodes. The Raft consensus protocol defines three roles: leader, follower, and learner. Initially, a leader is elected using Raft's leader election algorithm. The leader assumes the responsibility of coordinating all incoming requests to ensure data consistency across all server nodes. Follower nodes forward requests to the leader and await coordination instructions. Learner nodes include those that were disconnected from the network and have reconnected, or new members joining the network that need to synchronize with the leader. The integration of the Raft Consensus protocol into Etcd serves the purpose of ensuring fault-tolerant and consistent operations across the entire array of Etcd server nodes.

3. **Authentication and Authorization:**
   - **Performed by:** etcd Server Nodes, etcd Clients
   - **Description:** etcd v3 is using per-connection authentication instead of per-request authentication. Client needs to create a gRPC connection with etcd server. The purpose of this connection is only authentication and client needs to send its User ID and password. The authentication process will take place in API layer of etcd server for performance concern. Once authenticated, an unique JWT token will be issued back to the client and client could use the token to create a new connection and make API request.

### Goals

1. **Simplicity and User-Facing API**

   - Etcd is designed with the intention of offering a user-friendly distributed key-value store. The project provides a well-defined and easily accessible API for users to interact with the system seamlessly.

2. **Security and Resilience**

   - Etcd is intended to deliver a secure distributed key-value store. This is achieved by implementing automatic TLS with optional client cert authentication. The system is architected to be resilient under various attack scenarios, ensuring the integrity and confidentiality of stored data.

3. **Performance Optimization**

   - Etcd aims to be a fast distributed key-value store. The project incorporates various techniques to enhance software performance, providing users with a fast and responsive experience when interacting with the key-value store.

4. **Reliability**
   - Etcd is dedicated to providing a reliable distributed key-value store architecture. The system is properly distributed using the Raft consensus protocol, contributing to the system's stability and consistency.

### Non-goals

1. **Unlimited Data Storage:**

   - Etcd does not aim to prevent a party with a valid key from storing an arbitrarily large amount of data. While the project emphasizes efficiency and performance, it does not set explicit limits on the size of data that can be stored, recognizing that users may have varying storage needs.

2. **Server Overload Prevention:**

   - Etcd does not claim to prevent a party from overloading servers intentionally. While the system is designed to be resilient, users are expected to adhere to best practices for resource usage to ensure optimal performance and avoid intentional server overload scenarios.

3. **Complete Security Against All Attacks:**
   - Etcd does not claim to provide absolute security against every conceivable attack. While the project strives to implement robust security measures, it acknowledges the evolving nature of security threats. Users are encouraged to adopt additional security measures based on their specific security requirements.

## Self-assessment use

This self-assessment is created by the etcd team to perform an internal analysis of the
project's security. It is not intended to provide a security audit of etcd, or
function as an independent assessment or attestation of etcd's security health.

This document serves to provide etcd users with an initial understanding of
etcd's security, where to find existing security documentation, etcd plans for
security, and general overview of etcd security practices, both for development of
etcd as well as security of etcd.

This document provides the CNCF TAG-Security with an initial understanding of etcd
to assist in a joint-assessment, necessary for projects under incubation. Taken
together, this document and the joint-assessment serve as a cornerstone for if and when
etcd seeks graduation and is preparing for a security audit.

## Security functions and features

### Critical

1. **Transport Layer Security**

   - Etcd employs automatic TLS with optional client cert authentication to securely communicate between nodes and clients, ensuring confidentiality and integrity of the data. It is critical for securing sensitive data during transimission, protecting against unauthorized access and tampering.

2. **Raft Consensus Protocol**

   - Etcd utilizes the Raft consensus protocol for distributed coordination. Raft consensus protocol is critical for ensuring consistency and fault tolerance of the distributed system. The protocol maintains a consistent and replicated state across all distributed nodes, contributing to the overall avaliability and reliability of the key-value store.

3. **Per-connection Authentication Mechanism**
   - Etcd employs per-connection authentication to ensure both security and performance in user authentication. Clients initiate a connection with a User ID and password solely for the purpose of authentication. Upon successful authentication, the server issues a JWT token to the client. Subsequent connections and requests by clients are established and made using this token. This authentication mechanism holds critical importance for etcd as it governs access to the key-value store, guaranteeing that interactions with the system are conducted exclusively by authorized parties.

### Security Relevant

1. **Secure Defaults and Configurations**

   - Etcd employs secure default settings and configurations to minimize the risk of misconfigurations leading to security vulnerabilities.

2. **Audit Logging**

   - Etcd uses the zap library for comprehensive and robust logging capabilities, capturing security-related events and activities. This includes events such as the logging of a new user into the authentication system. The audit logging is important for monitoring and analysis, offering visibility into potential security issues and aiding in proactive threat detection.

3. **Role-Based Access Control**
   - Etcd supports Role-Based access control mechanisms to define and enforce roles for users, enabling fine-grained control over access to specific resources and operations, allowing administrators to define and manage user permissions effectively.

## Project compliance

- The etcd project has not been explicitly documented as meeting specific security standards such as PCI-DSS, COBIT, ISO, GDPR, or others. While etcd is designed with security in mind and follows secure development practices, the project's documentation does not explicitly claim compliance with specific external standards.

- However, it's essential to note that the security features provided by etcd, such as TLS encryption for communication, authentication mechanisms, and authorization controls, contribute to its ability to align with security best practices and potentially meet the requirements of various standards. Additionally, etcd is often used as a component within larger systems (e.g., Kubernetes), and the overall compliance of the system may depend on the configuration and practices of the entire deployment.

## Secure development practices

**Development Pipeline**

- **Continuous Integration and Continuous Delivery (CI/CD) Pipeline:** An automated pipeline that automates tasks like code linting, unit testing, integration testing, end-to-end testing, performance testing, and vulnerability scanning.
- **Contributor Commit Signing:** Contributors must sign their commits using Git's GPG signature feature to ensure authenticity and traceability.
- **Immutable and Signed Container Images:** Etcd container images are built using a reproducible and deterministic process and are signed to prevent unauthorized modifications.
- **Code Review Process:** All code changes undergo a rigorous code review process involving at least two reviewers to ensure correctness, adherence to style guides, security considerations, and overall design quality.
- **Automated Vulnerabilities Checks:** Etcd leverages various automated vulnerability scanning tools, such as Snyk and OSSFuzz, to proactively identify potential security issues in the codebase. These scans are integrated into the CI/CD pipeline, ensuring prompt detection and remediation of vulnerabilities.

**Communication Channels**

**Internal Communication:**
Etcd team members primarily utilize Slack for real-time discussions, GitHub for issue tracking and code collaboration, and regular team meetings for in-depth discussions and planning.

**Inbound Communication:**
Users and prospective users can reach the Etcd team through:

- **GitHub Issues:** Report bugs, request features, and ask questions directly within the project's GitHub repository.
- **etcd-dev Mailing List:** Engage in discussions, seek assistance, and stay informed about project updates by subscribing to the mailing list.

**Outbound Communication:**
Etcd actively engages with its user community through:

- **Blog Posts:** Share technical insights, announce new features, and provide tutorials on Etcd's usage.
- **Release Announcements:** Inform users about new releases, bug fixes, and security updates.
- **Conferences and Meetups:** Participate in relevant events to showcase Etcd's capabilities and connect with the community.

**Ecosystem**

Etcd provides a central distributed key-value store that enables coordination and consistency across microservices and distributed applications.

**Integration with Popular Tools:**

- **Kubernetes:** Etcd serves as the default distributed key-value store for Kubernetes, managing cluster configuration, service discovery, and leader election.
- **Docker:** Etcd can be used to store and manage Docker container configurations, facilitating orchestration and deployment of microservices.
- **Cloud Platforms:** Etcd is supported by major cloud platforms, such as AWS, Azure, and GCP, enabling seamless integration with cloud-based infrastructure.

**Example Scenarios**

Scenarios where Etcd's impact extends beyond its direct user base:

- **Service Discovery in Microservices Architectures:** Etcd enables microservices to discover and connect with each other dynamically, ensuring seamless communication and service orchestration.

- **Configuration Management in Distributed Systems:** Etcd stores and distributes configuration information across a cluster, ensuring consistent application behavior and fault tolerance.

- **Leader Election for Coordinated Actions:** Etcd facilitates leader election among nodes in a distributed system, enabling coordinated actions and preventing conflicts.

## Security issue resolution

**Responsible Disclosures Process**

**Private Disclosure Processes**

1. **Reporting**: Internal and external individuals discovering potential security issues are encouraged to report them promptly.

2. **Communication Channels**:

   - **Internal Reporting**: Team members or contributors can report internally through designated communication channels.
   - **External Reporting**: External parties can report issues by emailing the Product Security Committee (PSC) at [security@etcd.io](mailto:security@etcd.io).

3. **Private and Responsible Disclosure**: The etcd community encourages submitters to follow responsible disclosure practices, allowing the PSC to address and resolve the issue before public disclosure.

4. **Acknowledgment and Analysis**: Upon receiving a report, the PSC acknowledges it within 3 working days and initiates an analysis to understand the scope and severity of the reported security issue.

**Public Disclosure Processes**

1. **Immediate Reporting**: In case of a publicly disclosed security vulnerability, anyone aware of it should immediately inform the PSC via [security@etcd.io](mailto:security@etcd.io).

2. **Optional Private Handling**: The PSC may discuss with the reporter the possibility of handling the issue through a private disclosure process. If denied or impractical, the PSC proceeds promptly with the fix and release process.

3. **GitHub Interaction**: In extreme cases, GitHub may be approached to delete the issue, though this is typically unnecessary.

**Vulnerability Response Process**

**Reporting Responsibility**

- **Report Submission**: Any individual discovering a vulnerability can submit a report through the established communication channels.
- **Responsibility for Response**: The PSC is responsible for responding to and managing reported vulnerabilities.

**Reporting Process**

1. **Fix Team Formation (Within 24 hours of Disclosure)**:

   - The PSC rapidly identifies relevant engineers, forming the Fix Team, which includes maintainers and volunteers.

2. **Fix Development (1-7 days of Disclosure)**:

   - The PSC and Fix Team use the CVSS Calculator to assess severity and request a CVE.
   - The Fix Team develops the fix, and the PSC ensures the completion of relevant processes.

3. **Fix Disclosure (1-21 days of Disclosure)**:
   - The PSC coordinates the release, merging patches, and announcing new releases with CVE numbers and severity details.

**Incident Response**

1. **Triage and Confirmation (Immediate)**:

   - The PSC ensures the prompt triage of reported incidents, confirming the validity and severity of the vulnerability.

2. **Notification (Upon Confirmation)**:

   - Once a vulnerability or security incident is confirmed, the PSC notifies relevant stakeholders, including developers and maintainers.

3. **Patching/Update Availability**:
   - The Fix Team works on developing the necessary patches or updates, aiming to make them available within the established timelines.

**Contact Information**

- For security-related communications, contact the PSC at [security@etcd.io](mailto:security@etcd.io).

## Appendix

Etcd maintains a commendable track record of promptly and effectively addressing security vulnerabilities. The project's GitHub repository maintains a detailed history of reported issues and their corresponding fixes, indicating a strong commitment to maintaining a high level of security.

**Vulnerability Response Process**

Etcd adheres to a well-defined vulnerability response process that ensures timely and effective remediation of security flaws. Upon receiving a vulnerability report, the Etcd security team promptly investigates the issue, assesses its severity, and develops a fix or mitigation strategy. Once a fix is available, a public disclosure is made, including a detailed description of the vulnerability, its impact, and the mitigation strategy.

**Examples of Past Vulnerabilities**

Here are a few examples of past vulnerabilities that have been reported and addressed in Etcd:

- [**CVE-2021-44838:**](https://access.redhat.com/security/cve/cve-2021-44832) This vulnerability allowed an attacker to gain unauthorized access to data stored in Etcd. It was addressed in [Etcd v3.5.2](https://github.com/etcd-io/etcd/releases/tag/v3.5.2).
- [**CVE-2021-28235:**](https://nvd.nist.gov/vuln/detail/CVE-2021-28235) An authentication vulnerability found in Etcd v3.4.10 that allows remote attackers to escalate privileges via the debug function. This vulnerability was addressed in [Etcd v3.5.2](https://github.com/etcd-io/etcd/releases/tag/v3.5.2).
- [**CVE-2021-44839:**](https://access.redhat.com/security/cve/cve-2021-44832) This vulnerability allowed an attacker to cause Etcd to crash, leading to a denial-of-service (DoS) attack. It was addressed in [Etcd v3.5.2](https://github.com/etcd-io/etcd/releases/tag/v3.5.2).
- [**CVE-2022-27770:**](https://access.redhat.com/solutions/6021331) This vulnerability allowed an attacker to execute arbitrary code on Etcd servers. It was addressed in [Etcd v3.5.4](https://github.com/etcd-io/etcd/releases/tag/v3.5.4).
- [**CVE-2022-27771:**](https://snapcraft.io/install/etcd/rhel) This vulnerability allowed an attacker to gain unauthorized access to data stored in Etcd. It was addressed in [Etcd v3.5.4](https://github.com/etcd-io/etcd/releases/tag/v3.5.4).
- [**CVE-2022-27772:**](https://access.redhat.com/solutions/retbleed) This vulnerability allowed an attacker to cause Etcd to crash, leading to a denial-of-service (DoS) attack. It was addressed in [Etcd v3.5.4](https://github.com/etcd-io/etcd/releases/tag/v3.5.4).

Please refer to the official [Etcd Security Advisory](https://github.com/advisories/GHSA-gmph-wf7j-9gcm):

**CII practices**

The project adheres to the Core Infrastructure Initiative (CII) best practices, ensuring robust software security measures. (Link: https://www.bestpractices.dev/en/projects/3192) These practices encompass:

- **Documented Security Policy:** A comprehensive security policy outlines the project's approach to vulnerability management, software development, and incident response.

- **Vulnerability Management:** A well-defined process identifies, prioritizes, and remediates software vulnerabilities promptly, minimizing security risks.

- **Secure Software Supply Chain:** A robust supply chain security model ensures the integrity of all components and dependencies throughout the development lifecycle.

- **Secure Development Environment:** A controlled development environment fosters secure coding practices and minimizes the introduction of vulnerabilities.

- **Continuous Testing:** Rigorous testing and validation processes throughout the development cycle detect and address potential security flaws early on.

- **Secure Deployment:** A streamlined deployment process minimizes the window of opportunity for vulnerabilities to be exploited.

- **Incident Response:** A structured incident response plan ensures timely and effective mitigation of security breaches.

**Real-world Use Cases**

**1. Managing Distributed Configurations for a Microservices Architecture**

A large e-commerce company is using Etcd to manage the configuration of its microservices architecture. Etcd's distributed nature and strong consistency guarantees make it an ideal solution for this use case. By using Etcd, the company can ensure that all of its microservices are using the same configuration, regardless of where they are running. This has helped to reduce the number of configuration-related errors and improve the overall reliability of the system.

**2. Building a Highly Available Key-Value Store**

A startup is developing a highly available key-value store that needs to be able to handle millions of requests per second. The startup is using Etcd as the underlying storage layer for its key-value store. Etcd's high availability and scalability make it an ideal choice for this use case. By using Etcd, the startup can ensure that its key-value store can always meet the demands of its users.

**3. Implementing a Distributed Lock Service**

A software development team is using Etcd to implement a distributed lock service. Etcd's consensus mechanism and support for atomic operations make it an ideal solution for this use case. By using Etcd, the development team can ensure that their distributed applications can acquire and release locks in a consistent and reliable manner.

**Related Projects/Vendors**
| Feature | Etcd | ZooKeeper | Consul |
|-----------------------|----------------------------------------|-----------------------------------------|----------------------------------------------|
| **Primary Focus** | Distributed key-value store | Distributed coordination and configuration | Service discovery and configuration management |
| **Data Model** | Key-value pairs | Hierarchical tree | Key-value pairs, services, health checks |
| **Consistency** | Strong consistency | Eventually consistent | Eventually consistent |
| **Performance** | High performance | Good performance | Good performance |
| **Ease of Use** | Easy to use | Complex to use | Easy to use |
| **Ideal Use Cases** | Microservices architectures, highly available key-value stores | Distributed systems, configuration management | Service discovery, configuration management |

## Threat Modeling with STRIDE

**1. Spoofing**
**Threat:** An attacker could impersonate a legitimate Etcd member to gain unauthorized access to the cluster or disrupt its operation.

**Mitigation:**

- Implement strong authentication mechanisms: per-connection based authentication.
- tilize secure communication channels (e.g., TLS) for node-to-node and client-to-node communication.
- Regularly monitor and audit authentication logs.

**2. Tampering**
**Threat:** An attacker could modify Etcd data or code to alter its behavior or gain unauthorized access to sensitive information.

**Mitigation:**

- Use MVCC database (boltdb) to store multiple version of the data concurrently
- Use strong per-connection based authentication method to validate each request

**3. Repudiation**
**Threat:** An attacker could modify data or code and then deny that they made the changes.

**Mitigation:**

- Implement immutable and tamper-evident logging mechanisms.
- Use digital signatures to verify the integrity of log entries.
- Replicate data, code and logs among multiple machines in distributed system. Modifying data on a particular machine cannot result in presistent change.

**4. Information Disclosure**

**Threat:** An attacker could gain unauthorized access to sensitive information stored in Etcd, such as passwords or API keys.

**Mitigation:**

- Implement data encryption to protect sensitive information at rest and in transit. This can be achieved using AES 256-bit encryption or equivalent.

- Implement access control mechanisms to restrict access to sensitive data. This includes using role-based access control (RBAC) to grant users only the minimum level of access they need to perform their duties.

**5. Denial-of-Service (DoS)**

**Threat:** An attacker could overwhelm Etcd's resources with excessive requests, making it unavailable to legitimate users.

**Mitigation:**

- Implement resource quotas to limit the number of requests that a single member or client can make. This can help prevent an attacker from consuming all of Etcd's resources and causing a DoS attack.

- Implement rate limiting to slow down excessive requests. This can help prevent an attacker from flooding Etcd with requests and overwhelming its servers.

**6. Elevation of Privilege**

**Threat:** An attacker could gain unauthorized access to a higher level of privilege within the Etcd cluster, such as elevated permissions to modify data or control the cluster's operation.

**Mitigation:**

- Implement strong access control mechanisms to restrict access to privileged operations. This can include using RBAC and the principle of least privilege to grant users only the minimum level of access they need.

- Ensure that Etcd is configured securely, with all unnecessary ports closed and strong passwords used for all accounts.

- Regularly audit Etcd access logs to detect any suspicious activity that could indicate an unauthorized privilege escalation attack.
