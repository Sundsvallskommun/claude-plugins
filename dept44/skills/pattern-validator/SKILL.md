---
name: pattern-validator
description: "Patterns for adding field validation in dept44 microservices — from using built-in annotations to writing custom validators. Use this skill whenever you need to validate API model fields, check which dept44 validators already exist (@ValidUuid, @ValidMunicipalityId, @OneOf, @MemberOf, etc.), create a custom ConstraintValidator, compose multiple validators with OR/AND logic, validate complex objects with multiple violation messages, or write tests for validators. Also use when you see validation-related compilation errors, need to understand why enums aren't used in API models (use @OneOf instead), or are adding new fields to a request DTO and need to pick the right validation approach."
---

# Validator Pattern

dept44 provides a rich set of built-in validators in `dept44-common-validators`. Always check these first — writing a custom validator when a built-in one exists is wasted effort.

Source: `<repos>/dept44/dept44-common-validators/src/main/java/se/sundsvall/dept44/common/validators/annotation/` — `<repos>/` is the local clone root; if you don't already know it, use the `dept44-source` skill (its SKILL.md explains how to locate it), or browse `Sundsvallskommun/dept44` on GitHub.

## Reference Files

Read the relevant file(s) based on your situation.

| Topic | Reference file | When to read |
|---|---|---|
| Built-in validators + simple field validator | `references/builtin-and-simple.md` | Looking up existing validators or writing a basic single-field validator |
| Complex object validator | `references/complex-validator.md` | Validating an object with multiple fields, emitting multiple violation messages |
| Composite validator (OR/AND) | `references/composite-validator.md` | Combining existing validators without writing an implementation class |
| Testing validators | `references/testing.md` | Writing unit tests and `{Resource}FailureTest` integration tests for validators |
