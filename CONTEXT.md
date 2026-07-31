# Camille Document Editing

Camille document editing is the collaborative workspace context for changing document title, body, hierarchy, and access while keeping real-time editor state consistent across local tabs, offline storage, and connected collaborators.

## Language

**Editing Session**:
A browser-local period of interaction with one document screen where the current user can reverse their own recent title and body edits.
_Avoid_: Page history, version history, recovery draft

**Document Title**:
The document name stored as shared metadata in the collaborative Y.Doc.
_Avoid_: Heading

**Document Body**:
The BlockNote content stored in the collaborative Y.Doc.
_Avoid_: Page content, JSON content

**Undo Stack**:
The browser-local history of the current user's reversible edits during an Editing Session.
_Avoid_: Version history, audit log, rollback

**Redo Stack**:
The browser-local history of edits that were reversed from the Undo Stack and can be reapplied during the same Editing Session.
_Avoid_: Replay log, persisted redo

**Persisted Document State**:
The durable Yjs state stored locally or on the server so document content can recover and synchronize.
_Avoid_: Undo Stack, Redo Stack

**Document Edit**:
A reversible user action against either the Document Title or Document Body during an Editing Session.
_Avoid_: Body-only edit, title-only history entry

**Typing Batch**:
A group of continuous text input captured as one reversible Document Edit.
_Avoid_: Character history

**Structural Edit**:
A reversible change to document structure, such as creating, deleting, or moving a body block.
_Avoid_: Text input

**Formatting Edit**:
A reversible change to text styling, block type, checklist state, color, or another visible body property.
_Avoid_: Theme change, app setting

**UI State**:
Ephemeral interface state such as selection, cursor position, menu visibility, hover controls, or chrome visibility.
_Avoid_: Document Edit

**Document Command**:
An explicit user action that may change document lifecycle, hierarchy, or sharing in addition to the current document body.
_Avoid_: Document Edit

**Visible Placement Effect**:
The block-level insertion, removal, restoration, or repositioning that a Document Command causes in the current Document Body.
_Avoid_: Backend side effect, global page history

**Compensating Command**:
A follow-up command that reverses enough backend state to make an undone Visible Placement Effect valid.
_Avoid_: Local-only restore, page history restore

**Undo Shortcut**:
The keyboard command that reverses the most recent eligible Document Edit in the current Editing Session.
_Avoid_: Toolbar history button

**Redo Shortcut**:
The keyboard command that reapplies the most recently undone Document Edit in the current Editing Session.
_Avoid_: Toolbar history button

**Editable Session**:
An Editing Session where the document is not archived, collaboration is ready, and the current user has edit permission.
_Avoid_: Read-only session

**Editing Surface**:
The focused title input or body editor area where document edits are entered.
_Avoid_: Dialog input, menu search, global app control

**Session Undo Manager**:
The Yjs-level coordinator for the Undo Stack and Redo Stack during one Editing Session.
_Avoid_: ProseMirror history, body-only history

**Command Undo Metadata**:
In-memory data attached to a command-related undo entry so undo/redo can validate and run Compensating Commands.
_Avoid_: Persisted command history, Y.Doc command log

**Undo Failure Notice**:
A small toast shown when an undo or redo entry cannot complete after attempting a required Compensating Command.
_Avoid_: Empty-stack notice, read-only shortcut notice

## Relationships

- An **Editing Session** has at most one active **Undo Stack** and one active **Redo Stack**.
- An **Undo Stack** records the current user's **Document Edits** to the **Document Title** and **Document Body** in recency order.
- A **Redo Stack** replays reversed **Document Edits** in the opposite order.
- A new **Document Edit** after undo clears the **Redo Stack**.
- A **Session Undo Manager** coordinates the shared **Undo Stack** and **Redo Stack** for the collaborative Y.Doc.
- A **Typing Batch** is one **Document Edit** unless the user pauses, changes selection, or starts a distinct action.
- A **Structural Edit** is one **Document Edit**.
- A **Formatting Edit** is one **Document Edit** when it changes collaborative document state.
- **UI State** changes do not create **Document Edits**.
- A **Document Command** contributes a **Document Edit** only for its **Visible Placement Effect** in the current **Document Body**.
- A **Compensating Command** is required when undoing a **Visible Placement Effect** would otherwise leave the restored block pointing at invalid backend state.
- A **Compensating Command** must succeed before its dependent **Visible Placement Effect** is applied during undo or redo.
- **Command Undo Metadata** exists only in memory for the current **Editing Session**.
- **Command Undo Metadata** stores command type, target IDs, placement identity, and validation context rather than full document snapshots.
- **Persisted Document State** can survive reloads, but an **Undo Stack** and **Redo Stack** exist only for the current **Editing Session**.
- An **Undo Shortcut** and **Redo Shortcut** are the initial user-facing controls for the **Undo Stack** and **Redo Stack**.
- An **Undo Failure Notice** is shown only for failed compensated undo or redo entries.
- An **Undo Shortcut** and **Redo Shortcut** only apply during an **Editable Session**.
- An **Undo Shortcut** and **Redo Shortcut** are handled only from an **Editing Surface**.
- Executing undo or redo emits ordinary collaborative document updates and any required **Compensating Commands**.
- Losing edit permission ends the active **Editable Session** and invalidates stale undo/redo entries.
- Remote collaborator updates, socket replay, broadcast replay, initialization, and system synchronization do not become entries in the current user's **Undo Stack**.

## Example Dialogue

> **Dev:** "If I undo after a collaborator edits the same document, do I reverse their change?"
> **Domain expert:** "No. Undo applies to my **Editing Session** and should reverse my own **Document Title** or **Document Body** edits, not collaborator changes."
>
> **Dev:** "If I change the title, then add a body paragraph, what does undo reverse first?"
> **Domain expert:** "The body paragraph. **Document Edits** share one recency-ordered **Undo Stack** across title and body."
>
> **Dev:** "If I type a phrase continuously, should undo remove one character?"
> **Domain expert:** "No. Continuous typing is a **Typing Batch** and should undo as one **Document Edit** unless the user pauses or changes action."
>
> **Dev:** "If I move a subdocument block to trash, should undo restore only the block?"
> **Domain expert:** "Undo should restore the **Visible Placement Effect**, and because the block points to a child document, it also needs a **Compensating Command** so the restored child is active."
>
> **Dev:** "If I duplicate a subdocument block and then undo, can the duplicate remain active but hidden from the current body?"
> **Domain expert:** "No. Undoing the duplicate's **Visible Placement Effect** needs a **Compensating Command** so the duplicated subtree is not left active without its placement."
>
> **Dev:** "When I redo that duplicate, do we create another copy?"
> **Domain expert:** "Redo should restore the same duplicated subtree and placement when possible, so the **Redo Stack** reapplies the undone action instead of creating a new identity."
>
> **Dev:** "When I redo an undone archive, do we archive a replacement child?"
> **Domain expert:** "No. Redo should archive the same child subtree again and remove the same **Visible Placement Effect**."
>
> **Dev:** "Can my undo force a rollback over a collaborator's later change?"
> **Domain expert:** "No. A **Compensating Command** must validate the same target and current permission before it changes backend state."
>
> **Dev:** "If the Yjs document recovers after reload, does my undo history recover too?"
> **Domain expert:** "No. **Persisted Document State** can recover, but **Undo Stack** and **Redo Stack** are cleared when the **Editing Session** ends."
>
> **Dev:** "Do we need toolbar buttons for undo and redo?"
> **Domain expert:** "No. Start with **Undo Shortcut** and **Redo Shortcut** support; document toolbar actions remain separate."
>
> **Dev:** "Can a stale undo entry run after my edit permission is revoked?"
> **Domain expert:** "No. Undo and redo only run during an **Editable Session**."
>
> **Dev:** "Should the document undo shortcut run while a share dialog input is focused?"
> **Domain expert:** "No. Shortcuts are handled only from an **Editing Surface**, not unrelated app controls."
>
> **Dev:** "Will collaborators see the result when I undo?"
> **Domain expert:** "Yes. The **Undo Stack** is local, but executing undo produces normal collaborative updates."
>
> **Dev:** "Should body undo use ProseMirror history while title undo uses separate state?"
> **Domain expert:** "No. Use one **Session Undo Manager** over the collaborative Y.Doc so title, body, and command placement effects stay in recency order."
>
> **Dev:** "Where do we store the backend context needed to undo a duplicate or archive command?"
> **Domain expert:** "Use in-memory **Command Undo Metadata** for the current **Editing Session**, not persisted document state."
>
> **Dev:** "If I undo, then type something new, can I still redo the old undone edit?"
> **Domain expert:** "No. A new **Document Edit** branches history and clears the **Redo Stack**."
>
> **Dev:** "Can a subdoc block reappear before the child document is restored?"
> **Domain expert:** "No. The required **Compensating Command** must succeed before applying the dependent **Visible Placement Effect**."
>
> **Dev:** "Should we toast when undo has nothing to do?"
> **Domain expert:** "No. Use an **Undo Failure Notice** only when a compensated undo or redo entry fails after validation or command execution."
>
> **Dev:** "Does undo cover bold, heading changes, colors, and checklist toggles?"
> **Domain expert:** "Yes. Those are **Formatting Edits** when they change collaborative body state."
>
> **Dev:** "Does opening the slash menu or moving the cursor create an undo entry?"
> **Domain expert:** "No. Those are **UI State**, not **Document Edits**."

## Flagged Ambiguities

- "Undo like Notion" was resolved to mean browser-local **Editing Session** undo/redo across title and body edits, not durable page history or collaborator rollback.
- "Undo title and body" was resolved to mean one shared recency-ordered **Undo Stack**, not separate title and body histories.
- "Typing undo" was resolved to mean batched editor-native undo steps, not character-by-character reversal.
- "Command undo" was resolved to mean undoing visible block-level effects in the current document, not global page history across navigation.
- "Duplicate undo" was resolved to archive or otherwise deactivate the duplicated subtree when undo removes the duplicate block from the current document.
- "Duplicate redo" was resolved to restore the same duplicated subtree when possible, not create a fresh duplicate by default.
- "Archive redo" was resolved to archive the same restored child subtree again and remove the restored block placement.
- "Collaborative undo conflict" was resolved to fail gracefully and discard or disable the unsafe undo/redo entry instead of rolling back collaborator work.
- "Persistent undo" was rejected; undo/redo history is in-memory for one **Editing Session** only.
- "Undo controls" was resolved to keyboard shortcuts first, without visible toolbar buttons in the initial behavior.
- "Read-only undo" was rejected; undo/redo only applies while the document is editable and collaboration is ready.
- "Global undo shortcut" was rejected; undo/redo shortcuts are scoped to focused document editing surfaces.
- "Private undo result" was rejected; undo/redo results synchronize like ordinary document edits while undo history remains local.
- "Undo implementation layer" was resolved to a Yjs-level **Session Undo Manager**, not separate ProseMirror and title histories.
- "Command undo metadata" was resolved to in-memory session metadata, not a persisted Y.Doc command log.
- "Redo after branching edit" was rejected; new document edits after undo clear the redo stack.
- "Local-first compensated undo" was rejected; compensated undo/redo validates backend state before applying the visible effect.
- "Empty undo feedback" was rejected; only failed compensated undo/redo entries show a notice.
- "Formatting undo" was resolved to include formatting and block property changes that affect collaborative document state.
- "UI undo" was rejected; selection, cursor, menu, hover, and chrome visibility changes are not undoable document edits.
