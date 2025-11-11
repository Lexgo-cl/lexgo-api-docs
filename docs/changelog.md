# Changelog

All notable changes to the LexgoSign API will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.1] - 2025-11-10

### Added
- Email validation (2FA) API endpoints
- Comprehensive webhook event tracking for validations
- Evidence tracking for validation lifecycle

---

## [1.1.0] - 2025-11-01

### Added
- **Email Validation (2FA)**: Two-factor authentication via email verification codes
  - `POST /api/v1/envelopes/:envelope_id/recipients/:recipient_id/validations/send_code` - Send verification code
  - `POST /api/v1/envelopes/:envelope_id/recipients/:recipient_id/validations/verify_code` - Verify code
- **Validation Events**: Six new webhook events for email validation lifecycle
  - `recipient.validation_code_generated` - Code created
  - `recipient.validation_code_sent` - Email delivered
  - `recipient.validation_code_success` - Successful verification
  - `recipient.validation_code_failed` - Failed attempt
  - `recipient.validation_code_expired` - Code expired
  - `recipient.validation_max_attempts` - Too many attempts
- **Security Features**:
  - SHA256 code hashing (never stores plain text codes)
  - Constant-time comparison (timing attack prevention)
  - Rate limiting (5 codes/hour, 60-second cooldown)
  - Attempt limiting (3 attempts per code)
  - 30-minute code expiration
- **Evidence Tracking**: Complete audit trail for validation events
- **Custom Email Templates**: Template variables for validation emails

### Changed
- Settings API now supports validation configuration
- Evidence sheet includes validation events

### Security
- Implemented constant-time code comparison to prevent timing attacks
- Added cryptographic hashing for verification codes
- Enhanced rate limiting to prevent abuse

---

## [1.0.0] - 2023-01-01

### Added
- Initial API release
- **Envelopes API**:
  - `POST /api/v1/envelopes` - Create envelope
  - `GET /api/v1/envelopes/:id` - Get envelope details
  - `POST /api/v1/envelopes/:id/send_invitation` - Send invitation
  - `PUT /api/v1/envelopes/:id/void` - Cancel envelope
  - `GET /api/v1/envelopes/:id/evidence` - Get evidence sheet
- **Settings API**:
  - `GET /api/v1/settings` - Get configuration
  - `PUT /api/v1/settings` - Update configuration
- **Webhooks API**:
  - `GET /api/v1/webhooks` - List webhooks
  - `POST /api/v1/webhooks` - Create webhook
  - `GET /api/v1/webhooks/:id` - Get webhook
  - `PUT /api/v1/webhooks/:id` - Update webhook
  - `DELETE /api/v1/webhooks/:id` - Delete webhook
- **Webhook Events**:
  - Envelope events: `envelope.in_progress`, `envelope.success`, `envelope.voided`
  - Recipient events: `recipient.email_sent`, `recipient.email_received`, `recipient.email_opened`, `recipient.email_clicked`, `recipient.email_bounced`, `recipient.link_generated`, `recipient.in_progress`, `recipient.signed_all`, `recipient.rejected`
- **Authentication**: API key-based authentication
- **Multi-tenant Support**: Enterprise-level data isolation
- **Evidence Sheets**: Comprehensive audit trails with timestamps and IP addresses
- **Custom Email Templates**: Configurable email content and branding
- **Multi-language Support**: English and Spanish localization

---

## Version History

| Version | Release Date | Status | Notes |
|---------|--------------|--------|-------|
| 1.1.0 | 2025-11-01 | Current | Email validation (2FA) |
| 1.0.0 | 2023-01-01 | Stable | Initial release |

---

## Upcoming Features

### Planned for v1.2.0
- Phone number validation (SMS-based 2FA)
- ID document verification
- Biometric validation support
- Batch envelope operations
- Advanced search and filtering

### Under Consideration
- GraphQL API
- Real-time notifications via WebSockets
- Template management API
- Bulk recipient management
- Advanced reporting endpoints

---

## Deprecation Policy

When we deprecate an API feature:

1. **Announcement**: 6 months advance notice via email and changelog
2. **Documentation**: Clear deprecation warnings added to docs
3. **Parallel Support**: Old and new versions supported simultaneously
4. **Migration Guide**: Detailed upgrade instructions provided
5. **Sunset Date**: Feature removed after deprecation period

---

## Breaking Changes

### v1.1.0
- **No breaking changes**: All changes are backward compatible
- Email validation is opt-in (disabled by default)
- New webhook events are additive

### v1.0.0
- Initial release (no breaking changes)

---

## Security Updates

We take security seriously. Security updates are released as soon as fixes are available.

### Reporting Security Issues

**Do not report security vulnerabilities in public issues.**

Email security issues to: seguridad@lexgo.cl

We'll respond within 48 hours and provide a timeline for fixes.

---

## Support

For questions about changes:
- **Contact**: [Contact form](https://www.lexgo.cl/contacto)
- **Documentation**: [docs.lexgo.cl](https://docs.lexgo.cl)

---

## Related Links

- [Migration Guide](guides/email-validation.md)
- [API Reference](api/overview.md)
- [Webhook Documentation](api/webhooks.md)
