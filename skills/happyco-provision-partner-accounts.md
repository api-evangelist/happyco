---
name: Provision HappyCo partner accounts
description: Create and manage HappyCo accounts and their users on the fly with the Partner API.
api: grpc.happyco.com (happyco.manage.account_provisioning.v1 + happyco.manage.account.v1)
operations: [CreateAccounts, UpdateAccountStatuses, ListAccounts, UpdateAccounts, AddUsers, ListUsers]
---

# Provision HappyCo partner accounts

Partner integrations use the HappyCo Partner API to create and administer
accounts programmatically.

## Auth
HTTP Basic over gRPC to `grpc.happyco.com` (see `authentication/happyco-authentication.yml`).
Partner credentials are scoped to the account-provisioning surface.

## Steps
1. **Provision** — call `CreateAccounts`
   (`happyco.manage.account_provisioning.v1.AccountProvisioningService`) to create
   new HappyCo accounts on the fly. Carry your own `external_id` on the
   `IntegrationID` so you never maintain a separate HappyCo↔your-system ID map.
2. **Manage status** — call `UpdateAccountStatuses` to activate/suspend accounts
   you created.
3. **Administer** — with `happyco.manage.account.v1.AccountService`, call
   `ListAccounts` (accounts these credentials can manage) and `UpdateAccounts`.
4. **Seat users** — with `AccountUsersService`, call `AddUsers`, `ListUsers`,
   `UpdateUsers`, and `UpdateUserStatuses`.

## Rules
- Bulk methods are transactional; a failure rolls the whole call back.
- Page list responses with `happyco.type.v1.Paging` / `PagingResponse`.
