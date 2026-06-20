# Packet 03: Product Image Storage And Upload Safety

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packets 01-02 Approved; `P3-IMAGE-001` |
| Layer Ownership | Product Images owns image metadata and storage lifecycle; reads Catalog identity |
| Risk Review | Mandatory pre-code and post-test human review by security, Catalog, UX, and storage owners |
| Produces | Safe local image boundary, admin/public behavior, `evidence/03-completion.md` |

## Objective

Provide a local/free-first product-image capability whose metadata, validation, storage paths, authorization, and cleanup rules can later move to S3 without changing Catalog consumers.

## Read First And Prerequisites

- Approved Phase 0-2 file, authorization, audit, configuration, and Catalog contracts.
- Packets 01-02 and the resolved `P3-IMAGE-001` decision.
- Phase 3 roadmap image, security, failure, logging, testing, and approval sections.

## In Scope

- ProductImage metadata, ordered gallery, one primary image, optional variant association, alt text, lifecycle, and concurrency.
- Storage abstraction plus local adapter rooted outside source-controlled/public executable locations.
- Allowlisted JPEG, PNG, and WebP uploads; byte-signature and decoder validation; configurable size, dimensions, and decoded-pixel limits.
- Generated opaque filenames, normalized extension, safe path construction, atomic write/metadata behavior, orphan cleanup, and soft-delete workflow.
- Admin upload/manage/delete APIs and UI; public read/serve behavior for active images.

## Out Of Scope

- Support attachments, SVG, video, arbitrary documents, browser-trusted MIME validation, image CDN/transformation, malware service, S3 provisioning, or direct client-to-cloud upload.

## Ownership And Read-Only Contracts

- **Owns:** ProductImage metadata, storage key, display order, primary/active/deleted state, alt text, dimensions, media type, checksum, and cleanup state.
- **Reads:** Product/variant existence and admin authorization through approved interfaces.
- **Must not own/mutate:** Product/variant truth, Inventory, Cart, Orders, or arbitrary files outside its configured storage root.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `POST /api/v1/admin/products/{productId}/images` | `catalog.manage` | Multipart upload; validates content and returns metadata, never a filesystem path. |
| `PUT /api/v1/admin/products/{productId}/images/{imageId}` | `catalog.manage` | Updates alt text/order/primary/variant link with concurrency. |
| `DELETE /api/v1/admin/products/{productId}/images/{imageId}` | `catalog.manage` | Idempotent logical deletion and scheduled physical cleanup. |
| `GET /api/v1/products/{productId}/images` | Anonymous | Active public image projections only. |

Use Phase 0 Problem Details. Reject unsupported media with `415`, excessive payload with `413`, validation with `400`, stale writes with `409`, and concealed unauthorized resource access with the approved `404/403` rule.

## UI Acceptance States

- Admin gallery: loading, empty, successful upload, progress, invalid type/content/size/dimensions, duplicate checksum warning, storage failure, stale conflict, forbidden, and delete confirmation.
- Reordering and primary selection preserve stable layout and keyboard focus; image controls have accessible names.
- Public product detail shows stable placeholders for missing/failed images, meaningful alt text, ordered thumbnails, keyboard selection, and no broken internal path.

## Likely File Areas To Inspect

- Core ProductImage entity/value objects/storage interfaces and Catalog contracts.
- Infrastructure EF mapping, local storage adapter, validation/decoder, cleanup worker, configuration, migration.
- API multipart limits/endpoints/DTOs/OpenAPI and Web admin/public views.
- Unit, integration, API, storage-security, UI, and accessibility tests.

## Architecture And Prohibited Dependencies

- Core knows storage keys/interfaces, not local paths, ASP.NET upload types, EF, S3, or image libraries.
- API streams bounded content to the application service; controllers do not write files or DbContext directly.
- Never combine user filename with a path, trust extension/MIME alone, execute uploaded content, or serve storage directories with directory listing.
- Storage writes and metadata cannot share a database transaction; use staged write, metadata commit, finalization, and compensating cleanup.

## Database Changes And Migrations

Add or reconcile ProductImage with product ID, optional variant ID, opaque storage key, normalized media type, byte length, width/height, checksum, alt text, order, primary/active/deleted/cleanup state, version, and audit timestamps. Index product/order/active and variant; enforce one primary active image per product using a portable service rule plus the strongest reviewed provider constraint.

## Security, Logging, And Failure Rules

- Set conservative reviewed defaults, proposed as 5 MB, 8,000 pixels per dimension, and 40 million decoded pixels; configuration may only tighten without review.
- Decode in a bounded process/library, reject malformed/polyglot content and decompression bombs, strip unneeded metadata where practical, and never preserve supplied path/name.
- Require permission, MVC anti-forgery where applicable, rate/payload limits, concurrency, and audit for upload/update/delete.
- Log IDs, result, normalized type/size, checksum prefix, storage operation, and correlation ID; never image bytes, local absolute paths, personal metadata, credentials, or request bodies.
- A failed upload leaves no active metadata; a failed cleanup records retry state and never causes product deletion.

## Implementation Steps

1. Obtain mandatory pre-code approval for formats, limits, decoder, storage root, public serving, staging/finalization, cleanup, authorization, and exact routes/UI.
2. Define framework-neutral metadata/lifecycle/storage/validation contracts.
3. Add reviewed mapping, constraints, indexes, and migration.
4. Implement bounded local storage and content validation with traversal/atomicity tests.
5. Implement authorized application commands, compensation, audit, and exact APIs.
6. Implement accessible admin gallery and public product gallery states.
7. Add cleanup/retry behavior and failure injection.
8. Run all tests, record evidence, and obtain mandatory post-test human approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Content safety | Extension/MIME/signature mismatch; malformed/polyglot; SVG/executable; oversize/dimensions/pixel bomb. |
| Path safety | Traversal names; reserved names; collisions; storage-root escape; no absolute-path response. |
| Lifecycle | Upload success; metadata/storage failure compensation; primary/order; soft delete; cleanup retry. |
| Authorization | Anonymous/customer denied; permission/anti-forgery; concealed ID mismatch; audit. |
| UI | Progress/empty/error/conflict/delete/placeholder; keyboard/focus/labels/alt text. |
| Migration | Fresh apply; indexes/constraints; no duplicate Product/Variant ownership. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "ProductImage|I.*ImageStorage|multipart|ContentType|FileName|Path.Combine|catalog.manage" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Only validated allowlisted images can become active, and user filenames never determine storage paths.
- ProductImage owns metadata/lifecycle while Catalog is read-only and later S3 replacement remains behind an interface.
- All API/UI states, compensation, audit, cleanup, security, and accessibility tests pass.
- No filesystem path, image bytes, sensitive metadata, or request body enters logs/responses.
- Mandatory pre-code and post-test human approvals and evidence exist.

## Security Stop Conditions

Stop for missing human review, ambiguous storage root/public serving, extension-only checks, unsafe SVG/arbitrary upload, unbounded decode, path construction from user input, direct controller writes, authorization/audit bypass, exposed local paths, orphan cleanup without retry, or destructive migration.

## Rollback / Forward-Fix And Handoff

Back up the local database and storage root. Disable upload routes and retain metadata/storage keys on rollback; do not bulk-delete files. Forward-fix cleanup and metadata consistency with an audited reconciler. Hand the public image projection/storage abstraction to Packets 10-11 and future S3 migration planning.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 03 only after all prerequisite gates, Packets 01-02, P3-IMAGE-001, and mandatory pre-code human approval. Implement only ProductImage metadata/lifecycle, a bounded safe local storage abstraction, exact admin/public APIs, accessible gallery UI, compensation/cleanup, audit, and tests. Validate bytes with an approved decoder and generated storage keys; never trust filename/MIME, expose paths, add SVG/arbitrary files, or alter Catalog/Inventory truth. Add evidence/03-completion.md, run checks, and stop for mandatory post-test security/storage review.
```
