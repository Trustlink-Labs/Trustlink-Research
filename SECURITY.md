# TrustLink Labs Security Policy

TrustLink Labs develops and publishes open research and software for
identity-first, privacy-aware stablecoin payments and blockchain
settlement infrastructure.

This document explains how to report security concerns and where to find
the authoritative TrustLink Labs resources.

## Official TrustLink Labs Resources

If you reached this page while searching for TrustLink Labs, start with
these official resources:

- [TrustLink Labs](https://github.com/Trustlink-Labs)
- [TrustLink Labs Research](https://github.com/Trustlink-Labs/Trustlink-Research)
- [TrustLink Pay](https://github.com/bigdreamsweb3/trustlink-pay)
- [Transfer Identity Documentation](https://github.com/bigdreamsweb3/trustlink-pay/blob/main/docs/TRANSFER-IDENTITY.md)
- [Transfer Settlement Network Documentation](https://github.com/bigdreamsweb3/trustlink-pay/blob/main/docs/TSN.md)
- [Veil Privacy Protocol](https://github.com/bigdreamsweb3/veil-privacy-protocol-vpp)

The [TrustLink Labs Research README](https://github.com/Trustlink-Labs/Trustlink-Research#readme)
is the primary knowledge base for published research, protocol concepts,
architectural decisions, and related implementation repositories.

## Reporting a Vulnerability

Do not disclose exploitable security vulnerabilities publicly through
issues, discussions, pull requests, or social media.

To report a potential vulnerability:

1. Open this repository’s **Security** tab.
2. Select **Report a vulnerability** when private vulnerability reporting
   is available.
3. Include the affected component, reproduction steps, potential impact,
   and any suggested mitigation.

If private reporting is unavailable, create a public issue titled
**Security contact request** without including vulnerability details.
A maintainer can then provide an appropriate private communication channel.

## Scope

Security reports may concern:

- TrustLink Pay implementations
- Transfer Identity Protocol components
- Transfer Settlement Network components
- Privacy Receiving Unit designs or implementations
- Veil Privacy Protocol components
- Cryptographic, authorization, identity, settlement, or key-management weaknesses
- Documentation errors that could lead to unsafe implementation or deployment

This repository primarily contains architectural and technical research.
Implementation vulnerabilities should be reported in the repository
containing the affected code whenever possible.

## Research and Design Feedback

Not every technical concern is a vulnerability.

Architectural criticism, threat-model analysis, protocol feedback, and
research corrections may be submitted through the relevant repository’s
normal issue or discussion channels, provided they do not reveal an
immediately exploitable weakness.

## Response Principles

TrustLink Labs aims to:

- acknowledge credible reports
- evaluate affected components and potential impact
- coordinate remediation before public disclosure
- credit responsible reporters when appropriate and requested
- update research or implementation documentation when assumptions change

## No Security Warranty

Research publications may describe experimental, incomplete, or evolving
systems. Unless explicitly stated otherwise, they should not be treated as
audited production systems or as guarantees of security, privacy,
regulatory compliance, or financial safety.
