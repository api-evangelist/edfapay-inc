---
name: Onboard an EdfaPay merchant
description: Create a partner, merchant, branch and role in the EdfaPay management API.
api: EdfaPay Management API
host: https://revamp-api.edfapay.com
auth: X-API-KEY header
operations:
  - createPartner    # POST /api/v1/partner
  - createMerchant   # POST /api/v1/merchant
  - createBranch     # POST /api/v1/branches
  - createRole       # POST /api/v1/roles/role
  - searchMerchants  # GET  /api/v1/merchant/search
generated: '2026-07-19'
method: generated
source: openapi/edfapay-inc-revamp.json
---

# Onboard an EdfaPay merchant

Provision the Partner -> Merchant -> Branch hierarchy (see
data-model/edfapay-inc-data-model.yml).

## Steps

1. **Authenticate** with `X-API-KEY` (must be an ADMIN- or PARTNER-scoped key).
2. **Create the partner** (if new): `createPartner` (`POST /api/v1/partner`).
3. **Create the merchant** under the partner: `createMerchant`
   (`POST /api/v1/merchant`).
4. **Create one or more branches**: `createBranch` (`POST /api/v1/branches`).
5. **Define roles** for the merchant's users: `createRole`
   (`POST /api/v1/roles/role`).
6. **Verify** with `searchMerchants` (`GET /api/v1/merchant/search`).

> For public merchant self-onboarding with document upload, use the separate
> `POST https://api.edfapay.com/merchantonboarding/onboard/` endpoint
> (`multipart/form-data`, requires your `partnerId` and matching `partnerOrigin`).

## Rules

- Field formats (conventions/edfapay-inc-conventions.yml): phone E.164
  (`+9665XXXXXXXX`), country ISO-2 (`SA`), KSA IBAN must start with `SA`.
- Responses use the `ApiResponseObject { code, message, errorCode, data }`
  envelope; a `400` carries the validation reason in `errorCode`/`message`.
