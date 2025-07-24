
# ReguStack - Health Equipment Tracking Smart Contract

## Overview

This smart contract provides a secure, auditable system to **register**, **track**, **update**, and **validate** medical equipment throughout its lifecycle on the **Stacks blockchain**. It supports:

* Equipment registration & lifecycle phase updates
* Regulatory validations by approved authorities
* Timeline tracking
* Role-based access (admin vs authority)

---

## Table of Contents

* [Contract Structure](#contract-structure)
* [Constants](#constants)
* [Data Structures](#data-structures)
* [Traits](#traits)
* [Public Functions](#public-functions)
* [Read-only Functions](#read-only-functions)
* [Private Functions](#private-functions)
* [Error Codes](#error-codes)
* [Usage Flow](#usage-flow)
* [Security Considerations](#security-considerations)

---

## Contract Structure

The contract defines:

* Equipment lifecycle management via phase transitions
* Validation framework by regulatory authorities
* History tracking using timelines
* Authority approval mechanism by the admin

---

## Constants

### Equipment Phases

```clojure
EQUIPMENT_PHASE_PRODUCED   ;; u1
EQUIPMENT_PHASE_EVALUATION ;; u2
EQUIPMENT_PHASE_ACTIVE     ;; u3
EQUIPMENT_PHASE_SERVICED   ;; u4
```

### Validation Types

```clojure
VALIDATION_TYPE_FDA     ;; u1
VALIDATION_TYPE_CE      ;; u2
VALIDATION_TYPE_ISO     ;; u3
VALIDATION_TYPE_SAFETY  ;; u4
```

### Error Constants

```clojure
ERR_UNAUTHORIZED           ;; err u1
ERR_INVALID_EQUIPMENT      ;; err u2
ERR_PHASE_UPDATE_FAILED    ;; err u3
ERR_INVALID_PHASE          ;; err u4
ERR_INVALID_VALIDATION     ;; err u5
ERR_VALIDATION_EXISTS      ;; err u6
```

---

## Data Structures

### Maps

| Map                      | Key                               | Value                              |
| ------------------------ | --------------------------------- | ---------------------------------- |
| `equipment-data`         | `{equipment-id}`                  | `{owner, current-phase, timeline}` |
| `equipment-validations`  | `{equipment-id, validation-type}` | `{validator, timestamp, active}`   |
| `regulatory-authorities` | `{authority, validation-type}`    | `{approved}`                       |

### Variables

* `admin-principal`: Contract owner
* `time-sequence`: Incremental timestamp generator (simulated counter)

---

## Traits

```clojure
(register-equipment (uint uint) (response bool uint))
(update-equipment-phase (uint uint) (response bool uint))
(get-equipment-timeline (uint) (response (list 10 {phase: uint, timestamp: uint}) uint))
(add-validation (uint uint principal) (response bool uint))
(verify-validation (uint uint) (response bool uint))
```

---

## Public Functions

### `register-equipment(equipment-id, initial-phase)`

Registers new equipment with a valid phase (admin or PRODUCED).

* Fails if equipment already exists.
* Adds to timeline.

### `update-equipment-phase(equipment-id, new-phase)`

Updates the phase of the equipment.

* Only the owner or admin can do this.
* Appends to timeline (max 10 entries).

### `add-regulatory-authority(authority, validation-type)`

Admin-only. Approves a principal as an authorized validator for a specific validation type.

### `add-validation(equipment-id, validation-type)`

Validator-only. Adds validation entry if not already added.

* Ensures validator is approved.

### `revoke-validation(equipment-id, validation-type)`

Admin or original validator can revoke an existing validation (sets `active` to `false`).

---

## Read-only Functions

### `is-admin(sender)`

Checks if the sender is the contract admin.

### `verify-validation(equipment-id, validation-type)`

Returns `(ok true)` if validation exists and is active.

### `get-equipment-timeline(equipment-id)`

Returns last 10 phase changes with timestamps.

### `get-equipment-phase(equipment-id)`

Returns the current phase of the equipment.

### `get-validation-details(equipment-id, validation-type)`

Returns optional validation record, if exists.

---

## Private Functions

### `get-current-time()`

Simulates a timestamp by incrementing `time-sequence`.

### `is-valid-phase(phase)`

Ensures the given phase is one of the defined lifecycle stages.

### `is-valid-equipment-id(equipment-id)`

Valid range: `1` to `1,000,000`.

### `is-valid-validation-type(validation-type)`

Ensures it's one of the supported validation types.

### `is-regulatory-authority(authority, validation-type)`

Checks if the authority is approved for the given validation.

### `is-valid-authority(principal)`

Basic sanity checks for a validator principal.

---

## Error Codes

| Error                     | Meaning                          |
| ------------------------- | -------------------------------- |
| `ERR_UNAUTHORIZED`        | Action not allowed for caller    |
| `ERR_INVALID_EQUIPMENT`   | Equipment ID invalid             |
| `ERR_PHASE_UPDATE_FAILED` | Internal phase update failure    |
| `ERR_INVALID_PHASE`       | Invalid lifecycle phase          |
| `ERR_INVALID_VALIDATION`  | Invalid validation type          |
| `ERR_VALIDATION_EXISTS`   | Duplicate validation not allowed |

---

## Usage Flow

1. **Admin registers** the contract.
2. **Authorities are approved** using `add-regulatory-authority`.
3. **Equipment is registered** (by admin or users, if starting in `PRODUCED` phase).
4. **Phase updates** are performed by owner or admin.
5. **Authorities add validations** to the equipment.
6. **Timeline and validation status** can be queried at any time.
7. **Admin/validator can revoke** validation when needed.

---
