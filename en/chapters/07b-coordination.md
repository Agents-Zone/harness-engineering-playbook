# Replacing Interpersonal Coordination with Mechanisms

> 🚧 This section is under development.

Zhang's Agent refactored the payment module's interface. Li's Agent is still calling it with the old interface. Previously, Zhang would mention at standup "I am changing the payment interface today," and Li would hear it and know to hold off on touching related code. This informal coordination was nearly zero cost, relying on the human ability to hear something and adjust behavior accordingly. Agents have no such communication channel. They each faithfully execute their own instructions, completely unaware of each other's existence.

The alternative is to externalize coordination information from interpersonal networks into explicit mechanisms. Interface contracts define the interaction specifications between modules, and any modification to a contract must trigger notification and verification for all consumers. Shared state (which modules are currently being modified, which interfaces are about to change) is visible to all Agents. Change notification mechanisms automatically block other Agents from continuing development against an old interface when one party modifies a public interface. The design principle behind these mechanisms: take the coordination information that previously existed in people's heads and turn it into structured data that Agents can consume.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
