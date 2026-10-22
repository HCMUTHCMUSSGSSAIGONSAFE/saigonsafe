# MindSafe

**Built with TideCloak**

## Project overview

MindSafe is a privacy-focused mental health platform that empowers students to manage sensitive wellbeing records and share them with trusted clinicians or researchers through explicit, purpose-specific consent.

## 1. Vision & originality

Students may avoid seeking mental health support because they fear that sensitive personal information could be exposed to university administrators, employers, or other unauthorized parties.

MindSafe proposes a personal health-data vault where students can manage emotional wellbeing records and control how their information is shared. Access is intended to be limited by recipient, purpose, and duration rather than relying on a one-time consent checkbox.

## 2. Trust constraint insight

Traditional login systems and backend role-based access control can restrict application requests, but they do not automatically prevent privileged infrastructure operators from accessing sensitive plaintext data.

MindSafe aims to address this trust constraint through controlled authorization, encryption, and appropriate key management. The system must demonstrate that access restrictions are enforced beyond the user interface.

## 3. Value & impact

- **Students:** Greater control over sensitive wellbeing information.
- **Clinicians:** Access to specifically authorized records for care.
- **Researchers:** Access to appropriately minimized or aggregated data for approved research purposes.
- **Institutions:** Better understanding of overall wellbeing trends while reducing unnecessary exposure of individual records.

Research data must be designed to reduce the risk of identifying individuals.

## 4. Why TideCloak matters

MindSafe is designed to use TideCloak as part of its identity and access-control architecture rather than relying solely on conventional application login.

The project aims to demonstrate recipient-specific and time-limited authorization while protecting sensitive records from unauthorized access, including access attempts by database administrators.

These protections depend on the actual TideCloak integration, encryption design, key custody, and enforcement mechanisms. Connecting an AI coding agent to Tide's MCP server alone does not secure the deployed application.

The team will distinguish implemented and tested controls from planned capabilities.

## 5. Prototype & storytelling

The demo follows a student who wants mental health support without exposing their entire personal history.

1. The student creates a synthetic wellbeing record.
2. The student grants a designated clinician access to a specific record for a limited period.
3. An unauthorized recipient attempts to access the record and should be denied.
4. The team demonstrates the behavior after consent expires or is revoked.
5. The team tests whether a database administrator can recover plaintext from stored data without the required authorization and decryption capability.

The final demo will clearly distinguish actual security enforcement from simulated behavior.

## Setup

Setup instructions will be added after verifying the repository's application structure, dependencies, environment variables, and startup commands.

## Security note

This is a hackathon project. Do not use real mental health records until the necessary privacy, consent, and security safeguards have been implemented and verified.
