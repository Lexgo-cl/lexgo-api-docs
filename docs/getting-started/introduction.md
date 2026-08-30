# Introduction

## What is LexgoSign?

LexgoSign is a comprehensive **electronic signature platform** designed for businesses in Latin America and the United States. It provides legally binding digital signatures for contracts, legal documents, and business agreements with complete audit trails and compliance features.

## Key Features

### Electronic Signatures
- **Multi-signer support**: Create envelopes with multiple signers and approvers
- **Signature placements**: Define where each recipient should sign
- **Sequential workflows**: Control the signing order or do it in parallel

### Security & Compliance
- **Evidence sheets**: Complete audit trail with timestamps and IP addresses
- **Email validation (2FA)**: Require email verification before document access
- **Cryptographic verification**: SHA256 hashing and secure code storage
- **Multi-tenant isolation**: Enterprise-level data separation and security

### Integration Capabilities
- **RESTful API**: Clean, predictable endpoints
- **Webhook notifications**: Real-time event delivery
- **Custom templates**: Personalized signature flow and email communications
- **Multi-language support**: English and Spanish localization

## Use Cases

### Contract Management
Send employment contracts, NDAs, and business agreements for signature with automatic evidence collection.

### Legal Documents
Process legal forms, power of attorney documents, and court filings with legally compliant signatures.

### Onboarding Workflows
Streamline employee onboarding with multi-document signature packages and approval workflows.

### International Agreements
Support cross-border transactions with multi-language templates and timezone-aware tracking.

## API Design Principles

### RESTful
All endpoints follow REST conventions with standard HTTP methods (GET, POST, PUT, DELETE).

### JSON First
Request and response bodies use JSON format for easy integration.

### Authentication
API key authentication via `Authorization` header for security and simplicity.

### Idempotency
Safe operations (GET, PUT, DELETE) are idempotent for reliable integrations.

### Versioning
API versioned via URL path (`/api/v1/`) for backward compatibility.

## Coming Soon

We're constantly improving LexgoSign. Here are features currently in development:

### Templates API
- **Document templates**: Save frequently used documents as reusable templates
- **Field mapping**: Pre-define signature placements and custom fields
- **Template sharing**: Share templates across your organization

### Advanced Workflows
- **Approver roles**: Add approvers who review before signers
- **Conditional routing**: Route documents based on field values
- **Parallel approval**: Multiple approvers can review simultaneously

### Enhanced Security
- **Biometric signatures**: Face and fingerprint authentication
- **Video verification**: Record video consent during signing
- **Advanced encryption**: Additional encryption layers for sensitive documents

### Analytics & Reporting
- **Usage dashboards**: Track envelope volumes and completion rates
- **Performance metrics**: Monitor signing times and bottlenecks
- **Export capabilities**: Download reports in CSV and Excel formats

### Bulk Operations
- **Bulk send**: Send same document to multiple recipients
- **Batch processing**: Process multiple envelopes in one API call
- **CSV import**: Import recipient lists from spreadsheets

!!! info "Feature Requests"
    Have a feature request? [Contact us](https://www.lexgo.cl/contacto) to share your ideas.

## Getting Help

- **Documentation**: Browse our comprehensive guides and API reference
- **Support**: [Contact form](https://www.lexgo.cl/contacto) for bugs and feature requests

## Next Steps

1. [Set up authentication](authentication.md) - Get your API key
2. [Try the quick start](quick-start.md) - Create your first envelope
3. [Explore the API](../api/overview.md) - Learn about all endpoints
