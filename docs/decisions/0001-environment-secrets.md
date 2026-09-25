# Environment Configuration and Secrets

## Status

Accepted

## Context

SignalForge currently uses environment variables for database and Redis configuration, with local development defaults in Django settings.

During Milestone 1, Django also uses a hardcoded `SECRET_KEY` and `DEBUG=True`.

This is acceptable for local development during M1, but these settings must not be used as-is for a deployed environment.

## Decision

Before SignalForge is deployed, the Django `SECRET_KEY` must be supplied through an environment variable rather than being hardcoded in the source code.

The production environment must also use:

```text
DEBUG=False