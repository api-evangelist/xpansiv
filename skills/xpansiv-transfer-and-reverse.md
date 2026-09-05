---
name: xpansiv-transfer-and-reverse
description: Move environmental commodity holdings between counterparties on Xpansiv Connect, and use the two published reversal paths (cancel by sender, reject by recipient) while the transfer is still pending.
api: Xpansiv Connect API
base_url: https://connect.xpansiv.com/app/api/v1
generated: '2026-09-04'
method: generated
source: openapi/xpansiv-connect-openapi.yml + conventions/xpansiv-conventions.yml
operations:
  - getReferenceData
  - searchAccountPositions
  - createTransfers
  - searchTransfers
  - acceptTransfers
  - rejectTransfers
  - cancelTransfers
---

# Transfer holdings, and reverse one while you still can (Xpansiv Connect)

Unlike retirement, a transfer **is** reversible — but only while it is pending, and Xpansiv
publishes no time window for that. Treat the counterparty's `acceptTransfers` as the point
of no return, not a clock.

Authenticate as in `xpansiv-retire-environmental-commodity` (step 1).

## Sending

1. `getReferenceData` — `GET /reference-data`. Resolve the `ProgramCode`.
2. `searchAccountPositions` — `POST /portfolio/account/{AccountIdentifier}/position/action/search`.
   Confirm the holdings and quantities you intend to move.
3. `createTransfers` — `POST /transfers/account/{AccountIdentifier}/program/{ProgramCode}/action/create`.
   The transfer is created in a pending state awaiting the counterparty.
4. `searchTransfers` — `POST /transfers/account/{AccountIdentifier}/action/search`.
   Confirm exactly one transfer was created. **Always do this after a timeout.** There is
   no idempotency key on `createTransfers`, so a blind retry creates a second transfer,
   and two pending transfers that both get accepted move the holdings twice.

## Reversing

- **As sender**, `cancelTransfers` —
  `POST /transfers/account/{AccountIdentifier}/program/{ProgramCode}/action/cancel`.
  Use this to withdraw a transfer you initiated, including a duplicate you discovered in
  step 4.
- **As recipient**, `rejectTransfers` —
  `POST /transfers/account/{AccountIdentifier}/program/{ProgramCode}/action/reject`.
  Refuses an inbound transfer.

Both stop working once the transfer is accepted. Poll `searchTransfers` to know which
state a transfer is in before deciding.

## Receiving

`acceptTransfers` — `POST /transfers/account/{AccountIdentifier}/program/{ProgramCode}/action/accept`.
This is the irreversible half of the flow. Validate the inbound transfer against your own
expectations first; after acceptance the only way back is a new transfer in the opposite
direction, negotiated with the counterparty.

## The same shape on the other registries

The concept repeats across Xpansiv's acquired registry platforms with different names —
worth knowing if you integrate more than one:

- **NAR Registry**: `interaccountTransfer` (`POST /api/ledger/transfer`),
  `getPendingTransfers` (`GET /api/ledger/transfer/pending`), and
  `actionPendingTransfers` (`POST /api/ledger/transfer/pending/action`) to accept or
  reject. `subaccountTransfer` moves holdings within an account.
- **Optimal Transfer Position**: `initiateAccountTransfers`
  (`POST /api/ledger/{ledgerIdentifier}/transfer`), `getOpenTransferBatches`
  (`GET /api/ledger/{ledgerIdentifier}/transferBatch/open`), and
  `acknowledgeTransferBatches` (`POST /api/ledger/{ledgerIdentifier}/transferBatch/open/action`).

NAR and Optimal both accept a `correlationId` on the request — an opaque client value
echoed back in responses and error messages. It is a **correlation** identifier for
matching a response to a request, not an idempotency key; resending the same value does
not suppress a duplicate. They also return a `submissionId`, described in the contracts as
a "Submission identifier for issue investigation" — capture it and quote it to Xpansiv
support.
