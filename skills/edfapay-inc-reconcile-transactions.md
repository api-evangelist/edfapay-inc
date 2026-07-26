---
name: Reconcile EdfaPay transactions
description: Search EdfaPay financial transactions and pull full details and receipts for reconciliation.
api: EdfaPay Management API
host: https://revamp-api.edfapay.com
auth: X-API-KEY header
operations:
  - searchBranches        # GET /api/v1/transactions/filterTransaction
  - getTransactionDetailsById  # GET /api/v1/transactions/{id}/details
  - getTransactionReceiptById  # GET /api/v1/transactions/receipt/{id}
  - getPaymentGatewaySummary   # GET /api/v1/transactions/dashboard/payment-gateway-summary
generated: '2026-07-19'
method: generated
source: openapi/edfapay-inc-revamp.json
---

# Reconcile EdfaPay transactions

Use the EdfaPay management API to reconcile a period of payment-gateway activity.

## Steps

1. **Authenticate.** Send `X-API-KEY: <your key>` on every request. Keys are
   scoped to a caller type (ADMIN/PARTNER/MERCHANT/USER).
2. **Get the period summary.** Call `getPaymentGatewaySummary`
   (`GET /api/v1/transactions/dashboard/payment-gateway-summary`) to get totals
   to reconcile against.
3. **Search transactions.** Call `searchBranches`
   (`GET /api/v1/transactions/filterTransaction`) using page-number pagination
   (`pageNumber`, `pageSize`, `sortBy`, `sortDirection`). Iterate until the
   `PageObject.last` flag is true.
4. **Pull details.** For each transaction, call `getTransactionDetailsById`
   (`GET /api/v1/transactions/{id}/details`); pull `getTransactionReceiptById`
   (`GET /api/v1/transactions/receipt/{id}`) where a receipt is needed.
5. **Match settlement state.** Reconcile on the transaction `status`
   (`SETTLED`, `REFUND`, `DECLINED`) — see errors/edfapay-inc-decline-codes.yml.

## Rules

- Every response is wrapped in `ApiResponseObject { code, message, errorCode, data }`;
  read the payload from `data`. A `400` means a validation/reference error —
  inspect `errorCode` and `message`.
- Treat webhooks as advisory; the management API is the source of truth for
  settlement state.
