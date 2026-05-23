---
name: dto-entity-alignment
description: Architectural pattern for aligning remote server types with local database models. Use when designing data layers to ensure 1:1 mirroring in DTOs and consistent extension in Entities for local metadata.
---

# DTO-Entity Alignment Pattern

This architectural pattern ensures structural integrity between a remote server and a local client database while maintaining a clean separation of concerns.

## Core Principles

### 1. DTOs (Data Transfer Objects)

DTOs MUST be a 1:1 mirror of the server's data schema (e.g., Database Tables or API Types).

- **Purpose:** Act as a strict "Network Truth".
- **Naming:** Follow server-side naming conventions (e.g., snake_case if the server uses it).
- **Scope:** Include only data that is transmitted over the wire. No local-only flags, paths, or statuses.

### 2. Entities (Local Models)

Entities represent the local database schema and MUST "extend" the business data from DTOs with client-specific metadata.

- **Purpose:** Act as the "Application Truth" for persistence.
- **Composition:** Include all business fields from the DTO.
- **Extension Fields:** Add fields required for local lifecycle management:
  - **Sync State:** Tracks if data is pending, syncing, or failed.
  - **Offline Flags:** Marks data created or modified while offline.
  - **Local URIs:** Paths to locally stored media or files.
  - **Audit Metadata:** Client-side timestamps, device identifiers, or session IDs.

### 3. Mapping Logic

Always use explicit mapping functions to transform DTOs into Entities and vice-versa.

- **Direction (DTO -> Entity):** Copies business data and initializes default local metadata.
- **Direction (Entity -> DTO):** Strips local metadata to prepare a clean payload for the server.

## Recommended Workflow

1. **Mirror:** Create or update the DTO to match the latest server schema exactly.
2. **Extend:** Update the local Entity to include any new fields from the DTO.
3. **Default:** Provide default values for all new non-nullable local fields to maintain database stability.
4. **Map:** Update extension functions to handle the new fields during conversion.

## Benefits

- **Type Safety:** Changes on the server are caught immediately at the network layer.
- **Decoupling:** Backend schema changes do not force immediate complex database migrations if mapped correctly.
- **Accountability:** Local metadata enables robust auditing and offline-first support without cluttering server payloads.
