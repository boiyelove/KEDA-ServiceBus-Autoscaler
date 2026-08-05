# Architecture

## Context and scope

Provide cost-efficient, identity-based event scaling from zero for Service Bus workers on AKS.

## Components

Producers enqueue representative workloads; KEDA reads queue metrics using workload identity and scales workers. Cluster autoscaler expands a dedicated VMSS-backed pool, while dashboards correlate queue age, replicas, processing rate, failures, and spend.

The checked-in vertical slice owns request validation and deterministic plan
generation. A future Azure adapter owns authentication, retries, provider calls,
and reconciliation. The evidence plane in `infra/main.bicep` supplies a private,
identity-first baseline for logs and immutable-ready storage integration.

## Contract

Inputs are tenant-scoped JSON requests. Approved output includes project,
action, target, correlation ID, and an idempotency key. Any unknown action,
public access, secret material, stale evidence, or unapproved protected target
is rejected before an adapter can run. This separation makes denial behavior
testable without cloud credentials and prevents local demos from mutating Azure.

## Decisions

1. **Fail closed:** missing or unknown fields never downgrade into approval.
2. **Secretless:** only managed or workload identity reaches an adapter.
3. **Private by default:** both application requests and IaC reject public paths.
4. **Human authority:** production and break-glass changes require approval.
5. **Deterministic evidence:** canonical JSON supports replay and audit comparison.
