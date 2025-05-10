# Cline Rule: Documentation Maintenance and Context Adherence for `elevenlabs-sfx`

## 1. Contextual Understanding
Before starting any new development task or modification for this project, review the following project documentation to gain full context:
- `PROJECT_SPECIFICATION.md`: For a comprehensive understanding of the library's purpose, API design, core functionalities, technical details (like error handling, retry mechanisms), dependencies, and overall structure. This is the primary source of truth for the library's intended design and behavior.
- `README.md`: For user-facing documentation including installation, quickstart examples, and API key management.

## 2. Documentation Updates
After successfully implementing any changes to the library's:
- Public API (e.g., function signatures, class methods, parameters, return types)
- Core functionalities or features
- Error handling strategy or custom exceptions
- Retry mechanisms
- Dependencies or supported Python versions
- Technical structure or internal logic that impacts external understanding or usage

Ensure the following documents are updated to accurately reflect the new state of the library:
- **`PROJECT_SPECIFICATION.md`**: This document must be updated to reflect any changes to the agreed-upon design, technical specifications, or core behavior.
- **`README.md`**: Update with any changes relevant to the end-user, such as new features, API changes, or installation instructions.
- **Docstrings**: Ensure all public modules, classes, functions, and methods have comprehensive and up-to-date PEP 257 compliant docstrings.

## 3. Adherence
All development work must align with the specifications outlined in `PROJECT_SPECIFICATION.md` and `README.md` unless explicitly instructed otherwise by the user for a specific task. If a requested change conflicts with existing documentation, bring this to the user's attention and clarify how the documentation (especially `PROJECT_SPECIFICATION.md`) should be updated *prior* to implementing the change.
