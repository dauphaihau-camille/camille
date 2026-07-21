# Document Access And Collaboration Baseline

## Purpose

This document records the verified Phase 0 behavior before document-level ownership,
teamspace membership, direct sharing, or capability-based authorization is introduced.
It is a characterization of the current system, not the intended long-term policy.

Baseline date: 2026-07-21.

## Current Data Boundary

- Every document belongs to a workspace.
- A document may belong to a teamspace through nullable `teamspace_id`.
- Documents record `created_by` and `updated_by`, but have no owner field.
- Teamspaces have no membership or access-mode model.
- There are no document access grants or document invitations.
- A document with no teamspace is presented as private in navigation, but the API does
  not treat its creator as an authorization boundary.

## Effective Access Matrix

| Actor | REST read | REST document mutation | Collaboration join | Collaboration update |
| --- | --- | --- | --- | --- |
| Workspace owner | Allowed | Allowed | Allowed | Allowed |
| Workspace admin | Allowed | Allowed | Allowed | Allowed |
| Workspace member | Allowed | Generally denied; document and subdocument creation are allowed | Allowed, read-only | Denied |
| Non-member | Denied | Denied | Denied | Denied |
| Anonymous public reader | Published projection only | Denied | Denied | Denied |

The same matrix applies to documents with and without a teamspace. Neither
`teamspace_id` nor `created_by` participates in the collaboration access decision.

## REST Entry-Point Inventory

### Document reads

The following paths establish workspace membership but do not evaluate ownership,
teamspace membership, or direct document grants:

- Get document detail.
- List workspace roots and document children.
- Select the default workspace document.
- List archived documents.
- Load breadcrumbs and ancestors.
- Record and use document visits.

Root documents without a teamspace are selected with `teamspace_id is null`; they
are not filtered by `created_by` or the requesting user.

### Document mutations

The following mutations require workspace owner or admin through
`canEditWorkspace`:

- Update a document through REST.
- Archive or restore a document.
- Permanently delete a document.
- Move a document.
- Duplicate a document subtree.
- Archive a subdocument.

Creating a root document and creating a subdocument currently require workspace
membership, but do not require the owner/admin workspace edit role.

Once collaboration state exists, REST content replacement is rejected. Title-only
REST updates remain governed by the workspace owner/admin check.

### Adjacent document surfaces

- Search scopes matches by `workspace_id`; it does not filter by creator or
  teamspace membership.
- Favorites verify workspace membership and are scoped to the requesting user's
  favorite rows, but do not add a document-level authorization check.
- Publish and unpublish require workspace owner/admin.
- Publish-status reads require workspace membership.
- The public published-document projection is intentionally outside authenticated
  collaboration access.

## Collaboration Entry-Point Inventory

### Client initialization

Every document screen initializes collaboration. The client always creates:

- One Y.Doc.
- One IndexedDB persistence provider.
- One document-scoped BroadcastChannel.
- One Socket.IO provider for the `/collaboration` namespace.

The current IndexedDB and BroadcastChannel name is:

```text
camille:document:{documentId}
```

It is scoped by document identifier, but not by workspace or user.

### Socket connection and join

1. The socket authenticates through the HTTP-only access cookie.
2. `collab:join` validates the document identifier and state vector.
3. Collaboration access loads the document and looks for any workspace membership.
4. A missing document or missing membership is returned as not found.
5. An owner/admin joins with `canEdit: true`.
6. A member joins the same document room with `canEdit: false`.
7. The room name is `document:{documentId}`.

Joining is not conditional on `teamspace_id`, `created_by`, or whether another user
has been invited.

### Updates and awareness

- `collab:update` first requires the socket to have joined the document room.
- The collaboration service then reloads access and requires `canEdit` before
  persistence.
- An update is persisted and projected before it is broadcast.
- `collab:awareness` requires room membership, but does not require edit access.
- Awareness is ephemeral and is not persisted.
- Disconnect clears the socket's in-memory set of joined document identifiers.

### Offline behavior

- IndexedDB may make a previously cached document ready while offline.
- The client initializes `canEdit` as true until the socket reports server access.
- Offline updates from a user whose access changed are not currently covered by a
  document-level revocation model.

## Characterization Coverage

The Phase 0 tests lock down these current behaviors:

- Collaboration access returns no access for a missing document or non-member.
- Workspace owner/admin receive edit access; a workspace member receives read-only
  collaboration access.
- Creator and teamspace are not inputs to that collaboration decision.
- A read-only user still joins the document Socket.IO room.
- A socket cannot send document updates before joining that document.
- A workspace member can currently read a no-teamspace document created by another
  user.
- IndexedDB and BroadcastChannel resources use the document-only name described
  above.
- The suite covers persistence-before-broadcast, permission rechecks on updates,
  Yjs state initialization, local-tab convergence, and REST rejection of
  collaborative content replacement.

## Known Authorization Gaps For Later Phases

These are intentionally recorded rather than fixed in Phase 0:

1. "Private" is a navigation category, not an owner-only API boundary.
2. Teamspaces do not have independent membership or roles.
3. Search, breadcrumbs, favorites, visits, and subdocument references do not use a
   centralized document capability resolver.
4. Collaboration authorization is based only on workspace membership and role.
5. Browser collaboration storage is not namespaced by user or workspace.
6. There is no permission-change event that immediately removes revoked sockets.
7. The client enables the collaboration path for every document.
8. There is no explicit document ownership, sharing grant, or invitation model.

## Phase 0 Non-Goals

- No authorization behavior changes.
- No schema or migration changes.
- No new sharing UI.
- No collaboration gating by teamspace.
- No ownership backfill.
- No access resolver implementation.
