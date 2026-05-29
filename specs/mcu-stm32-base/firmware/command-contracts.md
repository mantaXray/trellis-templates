# Command Contracts

> Use this template when defining public device command APIs, UART command services, host-facing commands, or UI-triggered device writes.

## Scope / Trigger

Write a command contract when a task adds or changes:

- public command API signatures
- request/response payloads
- device TX ownership
- feature gates or optional hardware behavior
- host/UI/debug entry points that trigger device writes

## Signatures

List public APIs and callback entry points:

```c
app_result_t appDeviceCommandDoThing(const app_device_request_t *request);
app_result_t appDeviceCommandGetStatus(app_device_status_t *status);
```

## Contracts

- State the owning task or service.
- State whether the API is task-context only or ISR-safe.
- Define argument validation order.
- Define success meaning: queued, transmitted, acknowledged, executed, or snapshot updated.
- Define feature-disabled behavior and error code if hardware is optional.
- Define buffer ownership and lifetime for inputs and outputs.

## Validation & Error Matrix

| Condition | Required result |
|---|---|
| Null required pointer | explicit invalid-argument error |
| Unsupported feature or missing hardware | explicit unsupported/not-ready error |
| Queue full or TX busy | explicit busy/drop/retry result |
| Malformed payload | validation error before any TX side effect |
| Success | documented success meaning only |

## Good/Base/Bad Cases

- Good: valid request follows the owning command path and reports the correct success level.
- Base: existing behavior remains compatible when the feature is enabled.
- Bad: disabled or invalid commands must not start DMA, mutate state, or pretend the device responded.

## Tests Required

- API validation tests for null, length, range, unsupported, busy, and success paths.
- Integration or host tests for UI/host/debug command entry points when present.
- Regression tests showing disabled optional hardware has no TX side effect.

