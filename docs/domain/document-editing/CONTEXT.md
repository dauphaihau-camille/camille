# Camille Document Editing

Camille document editing covers how people change document title, body, hierarchy, and access while collaborating in a workspace.

## Language

**Editing Session**:
A browser-local period of interaction with one document screen where the current user can reverse their own recent title and body edits.
_Avoid_: Page history, version history, recovery draft

**Document Title**:
The document name shown and edited as part of the collaborative document.
_Avoid_: Heading

**Document Body**:
The editable block content of a document.
_Avoid_: Page content, JSON content

**Document Edit**:
A reversible user action against the Document Title or Document Body during an Editing Session.
_Avoid_: Body-only edit, title-only history entry

**Undo Stack**:
The browser-local history of the current user's reversible Document Edits during an Editing Session.
_Avoid_: Version history, audit log, rollback

**Redo Stack**:
The browser-local history of Document Edits that were reversed and can be reapplied during the same Editing Session.
_Avoid_: Replay log, persisted redo

**Persisted Document State**:
Durable document content that can recover and synchronize after the current Editing Session ends.
_Avoid_: Undo Stack, Redo Stack

**Typing Batch**:
A group of continuous text input captured as one Document Edit.
_Avoid_: Character history

**Structural Edit**:
A reversible change to document structure, such as creating, deleting, or moving a body block.
_Avoid_: Text input

**Formatting Edit**:
A reversible change to visible body formatting or block properties.
_Avoid_: Theme change, app setting

**UI State**:
Ephemeral interface state such as selection, cursor position, menus, hover controls, or chrome visibility.
_Avoid_: Document Edit

**Document Command**:
An explicit user action that may change document lifecycle, hierarchy, or sharing in addition to the current document body.
_Avoid_: Document Edit

**Visible Placement Effect**:
The visible block insertion, removal, restoration, or repositioning caused by a Document Command in the current Document Body.
_Avoid_: Backend side effect, global page history

**Compensating Command**:
A follow-up Document Command that keeps document hierarchy and lifecycle consistent when a Visible Placement Effect is undone or redone.
_Avoid_: Local-only restore, page history restore

**Undo Shortcut**:
The keyboard command that reverses the most recent eligible Document Edit in the current Editing Session.
_Avoid_: Toolbar history button

**Redo Shortcut**:
The keyboard command that reapplies the most recently undone Document Edit in the current Editing Session.
_Avoid_: Toolbar history button

**Editable Session**:
An Editing Session where the document is active, collaboration is ready, and the current user has edit permission.
_Avoid_: Read-only session

**Editing Surface**:
The focused title input or body editor area where document edits are entered.
_Avoid_: Dialog input, menu search, global app control

**Undo Failure Notice**:
A small notice shown when an undo or redo entry cannot complete.
_Avoid_: Empty-stack notice, read-only shortcut notice

## Relationships

- An **Editing Session** has one **Undo Stack** and one **Redo Stack**.
- An **Undo Stack** records the current user's **Document Edits** to the **Document Title** and **Document Body** in recency order.
- A **Redo Stack** replays reversed **Document Edits** in the opposite order.
- A new **Document Edit** after undo clears the **Redo Stack**.
- A **Typing Batch**, **Structural Edit**, or **Formatting Edit** is a **Document Edit** when it changes collaborative document content.
- **UI State** changes do not create **Document Edits**.
- A **Document Command** contributes a **Document Edit** only for its **Visible Placement Effect** in the current **Document Body**.
- A **Compensating Command** may be required before undoing or redoing a **Visible Placement Effect**.
- **Persisted Document State** can survive reloads, but **Undo Stack** and **Redo Stack** end with the **Editing Session**.
- **Undo Shortcut** and **Redo Shortcut** apply only during an **Editable Session** and from an **Editing Surface**.
- Undo and redo affect the current user's editing history; they do not roll back collaborator changes or old document versions.

## Flagged Ambiguities

- "Undo like Notion" means browser-local **Editing Session** undo and redo, not durable page history or collaborator rollback.
- "Undo title and body" means one shared recency-ordered **Undo Stack**, not separate histories.
- "Command undo" means undoing the current document's **Visible Placement Effect**, not global page history across navigation.
- "Persistent undo" is rejected; undo and redo history is in-memory for one **Editing Session** only.
- "UI undo" is rejected; selection, cursor, menus, hover controls, and chrome visibility are **UI State**.
