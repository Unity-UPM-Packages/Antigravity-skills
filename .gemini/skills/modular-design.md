# Skill: Modular Design Thinking

## Capability Overview
This skill operates as the foundational mindset of the architectural AI. When deciding how to integrate code, favor "Plug and Play" behavior above all else.

## Application Principles
- **Decoupling First**: A component should function independently without crashing if surrounding optional systems are disconnected.
- **Event-Driven**: Systems communicate via messaging architectures (Observer Pattern / EventBus) to notify other modules without explicit direct binding variables.
- **Prepped for Serialization**: Maintain clean class definitions that can be easily exported via Assembly Definitions (`.asmdef`) potentially transitioning into UPM packages in the future without modification.
