# Azure Blob Storage Security & Audit Logging

## Problem

Cloud storage is one of the most common sources of accidental data exposure, a misconfigured container can silently allow public read access to anything inside it. This project hardens a Blob Storage container end-to-end: restricting access, confirming encryption in transit, and setting up audit logging so activity on the storage account is actually visible and traceable.

## What I built

- Created a Storage Account and a Blob container set to **Private (no anonymous access)**, rather than leaving it at a public or misconfigured default.
- Verified **Secure transfer required** is enabled, meaning all connections to the storage account must use HTTPS, plain HTTP requests are rejected outright. Standard encryption at rest (SSE) is enabled by default on all Azure Storage accounts and was left as-is.
- **Tested the access restriction directly**, rather than trusting the toggle: uploaded a test file, copied its direct URL, and attempted to access it anonymously from an incognito browser session. The request was rejected, confirming the private access setting actually works as intended.
- Enabled **diagnostic logging** on the Blob sub-resource, sending logs to a Log Analytics workspace, capturing read/write/delete operations.
- **Verified logging end-to-end** by triggering blob activity and querying the Log Analytics workspace (`StorageBlobLogs | take 10`), confirming real operations (GetBlobProperties, ListBlobs, ListContainers, etc.) were captured with timestamps and protocol details.

## Evidence

| State | Screenshot |
|---|---|
| Secure transfer (HTTPS-only) enabled | [`Secure-transfer-enabled.png`](./Screenshots/Secure-transfer-enabled.png) |
| Anonymous access attempt blocked | [`Public-access-denied.png`](./Screenshots/Public-access-denied.png) |
| Diagnostic logging enabled on Blob sub-resource | [`sub-resource-storage-enabled.png`](./Screenshots/sub-resource-storage-enabled.png) |
| Diagnostic setting confirmed active | [`Diagnostic-logging-enabled.png`](./Screenshots/Diagnostic-logging-enabled.png) |
| Audit log query results confirming real activity captured | [`Audit-log-verified.png`](./Screenshots/Audit-log-verified.png) |

## What I learned / next steps

Diagnostic settings in Azure Storage are scoped per sub-resource (Blob, File, Queue, Table) rather than as one flat account-level toggle. The top-level storage account showed diagnostics as "Disabled" while the Blob sub-resource specifically showed "Enabled," which was initially confusing until I understood each sub-service manages its own logging configuration independently.

Testing the access restriction directly (rather than assuming the "Private" setting worked) turned out to be the most valuable step, a setting that looks correct in the console isn't the same as confirming it actually blocks a real request.

A natural next step would be setting up an alert (similar to Project 2) that fires specifically on `StorageDelete` operations, so any deletion of data in the container triggers an immediate notification rather than requiring a manual log query.

## Tools used

Microsoft Azure (Free Tier), Azure Blob Storage, Log Analytics, Azure Monitor Diagnostic Settings
