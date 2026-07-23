# Document Access Policy

## Purpose

This document describes the document-access policy implemented by the document
capability resolver and the APIs that consume it.

Last verified: 2026-07-23.

## Access Model

Document access is resolved to a capability set:

```ts
{
  accessScope: 'private' | 'shared' | 'teamspace';
  canView: boolean;
  canEdit: boolean;
  canManageAccess: boolean;
  permission: 'none' | 'view' | 'edit' | 'manage';
}
```

The resolver combines ownership, workspace role, teamspace role, direct document
grants, inherited ancestor document grants, and workspace-member access settings.

## Access Scope

| Scope | Rule |
| --- | --- |
| `teamspace` | The document belongs to a teamspace. |
| `shared` | The document has active grants, the actor has a direct grant, or workspace-member access is configured. |
| `private` | The document has no teamspace and no sharing signal. |

Teamspace scope takes precedence over shared scope.

## Permission Sources

| Source | Permission mapping |
| --- | --- |
| Private document owner | `manage` |
| Workspace owner/admin on teamspace document | `manage` |
| Teamspace manager | `manage` |
| Teamspace editor | `edit` |
| Teamspace viewer | `view` |
| Open teamspace default | `view` |
| Direct document grant `manage` | `manage` |
| Direct document grant `edit` | `edit` |
| Direct document grant `comment` | `view` |
| Direct document grant `view` | `view` |
| Ancestor document grant `manage` | `manage` |
| Ancestor document grant `edit` | `edit` |
| Ancestor document grant `comment` | `view` |
| Ancestor document grant `view` | `view` |
| Workspace-member setting `manage` | `manage` |
| Workspace-member setting `edit` | `edit` |
| Workspace-member setting `comment` | `view` |
| Workspace-member setting `view` | `view` |

When more than one source applies, the strongest permission wins:

```text
manage > edit > view > none
```

## Effective Access Matrix

| Context | Actor or source | View | Edit | Manage access |
| --- | --- | --- | --- | --- |
| Private document | Owner | Yes | Yes | Yes |
| Private document | Direct `manage` grant | Yes | Yes | Yes |
| Private document | Direct `edit` grant | Yes | Yes | No |
| Private document | Direct `comment` grant | Yes | No | No |
| Private document | Direct `view` grant | Yes | No | No |
| Private document | Workspace-member `manage` setting | Yes | Yes | Yes |
| Private document | Workspace-member `edit` setting | Yes | Yes | No |
| Private document | Workspace-member `comment` setting | Yes | No | No |
| Private document | Workspace-member `view` setting | Yes | No | No |
| Private document | No owner match, grant, or setting | No | No | No |
| Teamspace document | Workspace owner/admin | Yes | Yes | Yes |
| Teamspace document | Teamspace manager | Yes | Yes | Yes |
| Teamspace document | Teamspace editor | Yes | Yes | No |
| Teamspace document | Teamspace viewer | Yes | No | No |
| Open teamspace document | Workspace member without explicit teamspace role | Yes | No | No |
| Restricted teamspace document | Workspace member without explicit teamspace role | No | No | No |

## Resolver Notes

- The legacy `resolve(workspaceRole)` overload is retained for callers that still
  pass only a workspace role. It returns view access for all workspace roles and
  manage access for workspace owners/admins.
- `comment` is stored as a grant permission, but the current capability model maps
  it to `view` because the resolver does not expose a separate comment capability.
- Workspace owners/admins are not global document managers for private documents
  through the resolver unless another source grants access.
- Active direct grants on an ancestor document apply to descendants. The
  strongest applicable grant wins when direct and ancestor grants both apply.
- The resolver returns policy decisions from the context it receives. API behavior
  depends on callers loading and passing the relevant grant, setting, ownership,
  workspace, and teamspace data.

## API Surfaces

Current document-access work applies resolver-driven capabilities to document
reads, navigation, children, favorites, search, updates, and collaboration access.

Direct sharing operations include:

- Share a document with an existing workspace user.
- Change an existing user's permission by upserting their grant.
- Revoke a user's direct document access.
- List document collaborators.
- Get and update workspace-member access settings.

## Collaboration

Collaboration access is derived from the same document capability resolver:

- A user may join when `canView` is true.
- A user may send document updates when `canEdit` is true.
- Read-only users may join and receive collaboration state, but cannot persist
  updates.

## Storage

The access model is backed by:

- `documents.owner_user_id`
- `teamspaces.access_mode`
- `teamspace_members`
- `document_access_grants`
- `document_access_settings`

Direct grants are scoped to an existing workspace user and can be revoked without
deleting the grant row.
