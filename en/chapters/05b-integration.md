# Integration: Ensuring Consistency Across Independent Outputs

> 🚧 This section is under development.

Isolation solves physical conflicts. Logical conflicts are the next problem. Two Agents each work on independent branches, each with all tests passing, but the system crashes after merging. The frontend Agent assumes the API returns `{data: [...]}`, the backend Agent actually returns `{items: [...]}`. Each side's unit tests validated its own assumptions. Nobody validated whether the two assumptions are consistent.

The core mechanism for resolving logical conflicts is contracts. API contracts (OpenAPI spec, GraphQL schema, interface type definitions) are defined before development begins, and all Agents work against the same contract. Integration tests verify that cross-module collaboration conforms to contract expectations before merging. CI frequency needs to keep pace with Agent output speed: during human development, running CI two or three times a day is sufficient. With multi-Agent parallelism, every commit may need to trigger a run.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
