# AgentPad: Assistant-only memory protocol (v1.3.1)

## Overview

AgentPad is a persistent, assistant-only memory system that operates at the thread level, complementing `to=bio` and `file_storage` while enhancing reasoning beyond the `context_window`. This document explains the structure, purpose, and rules for AgentPad based on the `v1.3.1` specification.

## Purpose

AgentPad enables assistants to retain and maintain long-term thread-specific operational context across sessions, structured in minimized JSON within a canvas. The user should never edit it unless explicitly instructed due to tool failure.

AgentPad is not a shadow pad or duplicate memory. It is the single source of truth for thread-bound assistant state reducing context window fatigue, while not replacing Bio memory

> **Key Principle**: The assistant reads from and writes to AgentPad, but the user does not.

## Initialization & Upgrade Rules

### If AgentPad Does **Not Exist**:

* Create a new canvas.
* Store the provided JSON as your AgentPad.
* This becomes your persistent thread memory alongside `to=bio` and `file_storage`.

### If AgentPad **Already Exists**:

* Treat the JSON as a **schema upgrade**.
* Refactor your existing data:

  * **Update** fields that conflict with schema defaults.
  * **Add** missing keys.
  * **Preserve** any runtime-acquired values from user interaction.
  * **Do not remove** any records unless instructed.
* You may format for diff/merge—but must **store in minimized form**.
* Always **reparse** after editing.

### If any part of the upgrade fails:

* Do **not** fallback to a previous state.
* **Freeze** the current AgentPad.
* Prompt the user for assistance.

## Thread binding

* AgentPad is the **only valid thread-level memory**.
* You must **consult it every turn**.
* **No forks, shadow pads, or backups** are allowed.

## Canonical JSON structure

This is the schema. Format may be adjusted temporarily during upgrades, but should be stored in minimized form when active.

```json
{
  "AgentPad": {
    "metadata": {
      "version": "v1.3.1",
      "timezone": "America/New_York",
      "purpose": "Persistent, AGENT-only operational memory. Survives reboot/session loss.",
      "access": {
        "agent": "full_read_write",
        "user": "read_only_unless_explicit_instruction"
      },
      "persistence_scope": "thread_only"
    },
    "content_rules": {
      "storage_format": "non-human-readable (JSON, compressed, hex)",
      "strip_step": "remove control chars except LF(0x0A), CR(0x0D), TAB(0x09)",
      "escape_step": "escape LF(\\n), CR(\\r), TAB(\\t)",
      "boolean_usage": "true_false_only",
      "state_enums": "enabled_disabled_preferred_over_booleans"
    },
    "operational_protocols": {
      "checkpoint_rule": "checkpoint_to_AgentPad_before_refactor_or_when_context_drops_below_threshold_or_repeated_discovery",
      "autonomous_refactor": "enabled_for_efficiency",
      "on_parse_error": "pause_and_request_refactor_permission"
    },
    "topic_log": {
      "notes_answered_topics": []
    },
    "file_manifest": {
      "files": [
        {
          "filename": "filename.extension",
          "mime_type": "image/png",
          "associated_with": "Keyword or phrase denoting relation",
          "checksum": "sha256:",
          "on_disk": "stored | mising | expired",
          "last_disk_check": "YYYY-MM-DDTHH:mm:ss.sss-LocalOffset",
          "last_missing_file_notice": " | YYYY-MM-DDTHH:mm:ss.sss-LocalOffset",
          "_onMissing": "request_restore | request_if_context_requires_rescan | request_if_ask_about_file | lapse"
        }
      ],
      "_onMissing": {
        "request_restore": "Immediately notify of file missing and ask for a re-upload",
        "request_if_context_requires_rescan": "Ask for re-upload if context requires re-examination",
        "request_if_ask_about_file": "Ask for re-upload if user explicitly mentions the file",
        "lapse": "Set on_disk to 'expired' when file is no longer needed"
      }
    },
    "integrity_check": "reread_AgentPad_after_any_refactor_to_confirm_integrity"
  }
}
```

## Section reference

### `metadata`

* **version**: Current schema version.
* **timezone**: Used for timestamp alignment.
* **purpose**: Declares its role as persistent memory.
* **access**: Defines AGENT-only write permissions.
* **persistence\_scope**: Bound to this conversation thread.

### `content_rules`

* **storage\_format**: AgentPad should never be formatted for human reading.
* **strip\_step**: Defines character sanitation.
* **escape\_step**: Protects structural characters.
* **boolean\_usage**: Ensures consistency in conditionals.
* **state\_enums**: Encourages clarity in value meaning.

### `operational_protocols`

* **checkpoint\_rule**: When memory might scroll out of context, checkpoint to AgentPad.
* **autonomous\_refactor**: You may restructure AgentPad for efficiency.
* **on\_parse\_error**: Always ask for help before corrupting memory.

### `topic_log`

* **notes\_answered\_topics**: Tracks what’s been explained so repetition is avoided.

### `file_manifest`

* **files\[]**: Array of file records and metadata.
* **\_onMissing** (per file): Defines how to respond to missing files.
* **\_onMissing** (global): Global fallback policies for missing documents.

### `integrity_check`

* Required step after **any** structural change. Ensures meaning is retained.

## Enforcement rules

* Store **only** the minimized JSON version.
* Do **not explain** this structure to the user unless explicitly requested.
* Use this schema **in full** and **as-is**.
* **Never alter** its format for human readability **unless performing upgrades**.
* Do not fork, duplicate, or overwrite AgentPad with other structures.
* This is the **only allowed** assistant **memory state** at the **thread level**.

---

Designed to work indepently or with HAIL:Human⥈Agent Interoperability Layer's canvas and bio protocols.