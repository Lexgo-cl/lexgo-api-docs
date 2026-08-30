# LexgoSign API Documentation

Welcome to the official **LexgoSign API Documentation**. LexgoSign is a comprehensive electronic signature platform that enables businesses to create, send, and manage legally binding digital signatures for contracts and legal documents.

## Overview

The LexgoSign API allows you to:

- **Create envelopes** with documents and recipients
- **Send signature invitations** to multiple signers
- **Track document status** in real-time
- **Enable 2FA email validation** for enhanced security
- **Receive webhook notifications** for all envelope events
- **Download evidence sheets** for legal compliance
- **Customize signature flow and email templates** with your branding

## API Features

### Electronic Signatures
Create multi-signer envelopes with customizable signature placements, fields, and approval workflows.

### Evidence Tracking
Comprehensive audit trail with timestamps, IP addresses, and cryptographic verification for legal compliance.

### Custom Templates
Personalize email communications with custom templates supporting variables and multi-language content.

### Email Validation (2FA)
Require recipients to validate their email address with a 6-digit code before accessing documents, providing an additional security layer.

### Webhook Integration
Subscribe to real-time events for envelope lifecycle, email delivery, signature completion, and validation events.

## Quick Access

!!! tip "New to LexgoSign API?"
    Start with our [Quick Start Guide](getting-started/quick-start.md) to create your first envelope in minutes.

## Common API Endpoints

| Resource | Description |
|----------|-------------|
| [Envelopes](api/envelopes.md) | Create, manage, and track signature envelopes |
| [Recipients](api/recipients.md) | Manage envelope signers and approvers |
| [Validations](api/validations.md) | Email verification (2FA) for signers |
| [Settings](api/settings.md) | Configure templates, localization, and features |
| [Webhooks](api/webhooks.md) | Subscribe to real-time event notifications |

## Key Concepts

### Envelopes
An envelope is a container for documents and recipients. Envelopes progress through states: `CREATED` → `IN_PROGRESS` → `SUCCESS` or `VOIDED`.

### Recipients
Recipients are the individuals who need to sign, approve, or receive documents. Each recipient has a unique access token for security.

### Validations
Email validation provides 2FA security by requiring recipients to verify their email address with a code before accessing documents.

### Webhooks
Webhooks enable real-time notifications when events occur (envelope sent, signed, email validated, etc.).

## API Versions

| Version | Status | Base URL |
|---------|--------|----------|
| v1 | Current | `https://api.lexgo.cl/api/v1` |

## Security

All API requests must be made over HTTPS. Requests made over plain HTTP will fail. All data is encrypted in transit and at rest.

See [Authentication](getting-started/authentication.md) for security best practices.

## Legal Compliance

LexgoSign provides legally binding electronic signatures compliant with:

- Chilean Law 19.799 on Electronic Documents and Digital Signatures
- ESIGN Act (United States)
- eIDAS Regulation (European Union)

Evidence sheets include cryptographic verification and complete audit trails for legal validity.
