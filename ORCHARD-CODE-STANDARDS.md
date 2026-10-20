# Orchard Code Standards

Orchard is a community-developed Wayland compositor and desktop environment written primarily in C. These standards apply to Orchard-owned code across the project, including OrchardWM and future components such as OrchardShell, OrchardSession and OrchardUI.

**Read this document before writing or submitting code.** The goal is consistent, understandable, maintainable software, not perfection. Contributors and reviewers should treat each other respectfully. Performance, correctness, security and accessibility matter more than making code superficially short.

Some components may eventually need additional rules. Their repository-specific standards can extend this document, but should not silently contradict it.

## 1. File headers, licensing and attribution

Every Orchard-owned source file, including headers, should include a concise file header containing its license identifier, original author and creation date, and a brief description of its purpose.

Example (replace the license placeholder with the SPDX identifier adopted by the relevant Orchard repository):

```c
// SPDX-License-Identifier: <PROJECT_LICENSE>
// AmilieDev; 20/10/2026
// Handles an Orchard component's lifecycle.
```

For substantial contributions, credit additional contributors beneath the original attribution where appropriate. Do not remove existing copyright or attribution notices. Keep descriptions short unless the file genuinely requires more explanation.

**The existing standards file says `GPL-V2.0`, but that shorthand does not establish whether the project intends GPL-2.0-only or GPL-2.0-or-later.** The repository must choose and publish its actual license before contributors substitute a definitive SPDX identifier. This document does not make that decision.

Use comment syntax appropriate to the file's language. Do not impose Orchard's headers on vendored or third-party code when doing so would conflict with its license or upstream conventions.

## 2. Naming conventions

Use **`snake_case`** for Orchard-owned C variables, functions, fields and file names. Use **`UPPER_SNAKE_CASE`** for macros and compile-time constants.

```c
int active_window_count = 0;

void update_window_focus(void);

#define MAX_RESTART_ATTEMPTS 3
```

Avoid `camelCase`, `PascalCase` and unnecessary leading or trailing underscores in Orchard-owned identifiers. Public symbols should use a consistent component-specific prefix to avoid collisions, with the exact prefix documented by the component.

Do not rename external library identifiers or system API names merely to match Orchard's conventions. Follow the conventions required by Wayland, wlroots, POSIX and other dependencies when using their APIs. Protocol-generated identifiers are not governed by Orchard's naming rules.

## 3. Indentation and formatting

**Tabs are mandatory for indentation. Spaces must not be used as indentation in Orchard-owned code.** Spaces remain acceptable for alignment where needed, provided the resulting formatting is readable and consistent.

Additional requirements:

- No trailing whitespace.
- End every text file with a newline.
- Put unrelated statements on separate lines.
- Avoid unnecessarily long lines.
- Use blank lines to separate logical sections.
- Follow the existing brace style in the file; be consistent within a component.
- Do not reformat unrelated code in a focused pull request.

Generated files, vendored sources and formats with their own strict whitespace requirements are exceptions. Do not modify generated code by hand merely to satisfy these rules.

## 4. C correctness and memory ownership

OrchardWM is a compositor. Memory errors and lifetime mistakes can terminate the desktop session, so C code must make ownership and cleanup understandable.

- Clearly establish who owns allocated memory, file descriptors and other resources.
- Check allocation failures and meaningful error returns.
- Free or release resources on every applicable failure path, not just the successful path.
- Do not dereference pointers after the object they refer to has been destroyed.
- Avoid hidden global state when ownership can be made explicit.
- Prefer straightforward control flow over clever but fragile shortcuts.
- Treat compiler warnings as problems to investigate, not messages to silence without justification.

Where a function has non-obvious ownership rules, document whether it borrows, transfers or retains a resource.

## 5. Wayland and wlroots conventions

OrchardWM uses wlroots and Wayland. Contributors must understand the lifetime and event-driven behaviour of the objects they touch.

- Use APIs that match the wlroots version supported by the repository. Do not copy examples from incompatible releases without checking them.
- Register and remove listeners carefully, especially when a surface, output, seat or window is destroyed.
- Keep compositor state internally consistent when clients disconnect unexpectedly.
- Validate protocol state and handle failures without assuming clients behave correctly.
- Avoid blocking operations in the compositor's event loop.
- Follow the documented Wayland protocol requirements rather than inventing client behaviour.

Milestone A is a **nested, basic floating-window compositor**. Changes should support that scope unless a maintainer has explicitly agreed to expand it.

## 6. Component boundaries and public APIs

Orchard is planned as multiple cooperating components, not one giant executable.

- Keep OrchardWM's compositor responsibilities separate from shell UI and optional extensions.
- Prefer documented Wayland protocols for compositor-facing interactions and appropriate service interfaces for other desktop functions.
- Do not give third-party extensions direct access to compositor internals.
- Distinguish private implementation details from interfaces intended for other components or contributors.
- Do not promise API stability for an experimental interface unless its maintainers have explicitly designated it stable.

These are architectural expectations for future development, not requirements to implement OrchardShell or extension support during Milestone A.

## 7. Security and privacy

Orchard may eventually handle sensitive desktop operations, so contributors must consider security from the beginning.

- Treat external input and client requests as untrusted.
- Validate bounds, lengths, state transitions and permissions before performing operations.
- Avoid granting processes more privileges or information than necessary.
- Do not assume a cryptographic signature proves code is safe.
- Never treat an extension's self-declared identity as authorization for privileged operations.
- Do not log secrets or unnecessarily expose user data in diagnostics.

Future privileged extension APIs must enforce permissions at the authority performing the operation, not solely in the user interface that grants them.

Report suspected security vulnerabilities privately through the repository's designated security reporting channel when one is available, rather than publishing exploitable details in an issue.

## 8. Errors, logging and recovery

Failures must be diagnosable without producing misleading success messages.

- Handle recoverable errors explicitly.
- Include useful context in diagnostics without dumping sensitive information.
- Clean up resources on failed initialization and shutdown.
- Do not introduce infinite retry loops or repeated failures that overwhelm the session.
- Keep crashes of optional components isolated from OrchardWM wherever the architecture allows.

Orchard's long-term design calls for automatic shell-component recovery with a known-good fallback. That is **not** a Milestone A implementation requirement.

## 9. Testing and verification

Test the behaviour affected by your change before submitting it. Describe what you tested and what you could not test.

For Milestone A compositor work, relevant checks include:

- Starting OrchardWM nested inside an existing Wayland session.
- Launching and displaying a Wayland client.
- Keyboard and pointer handling.
- Floating-window focus, movement and resizing, where implemented.
- Client disconnects and window destruction.
- Repeated startup and clean shutdown without leaving unwanted processes or resources.

Use available compiler diagnostics and memory-checking tools where practical. Do not claim tests passed unless they were actually run. A nested compositor reduces risk to the host desktop but does not guarantee complete isolation.

## 10. Pull requests and commit messages

Use a descriptive title with a short category prefix, preserving the original project's preference for titles such as `Fix:`.

Examples:

- `Fix: Release listeners when a window is destroyed`
- `Feat: Add basic pointer focus handling`
- `Docs: Clarify nested compositor testing`
- `Refactor: Separate output initialization from startup`

Avoid titles such as `Updated files`, `Changes` or unrelated text. Explain **what changed, why it changed, and how it was tested** in the pull request description.

Keep pull requests focused. Avoid unrelated reformatting or combining unrelated features merely because they touch the same component.

## 11. AI-assisted contributions

Orchard's public project scope discourages AI-generated contributions but allows limited assistance, including documentation support and help understanding or improving code. **Purely AI-written commits are not accepted.** Contributors remain responsible for understanding, reviewing and testing everything they submit.

This standards document does not override the project's public AI contribution policy. Read the applicable repository contribution guidance before submitting work.

## 12. Review culture

Contributors will make mistakes, and reviewers will too. Be clear, constructive and specific when requesting changes. The aim is code that other people can understand and safely maintain.

Formatting rules are important, but they do not excuse hostile reviews. Where correctness, security, accessibility or performance conflicts with a stylistic preference, discuss the trade-off openly and document the decision.

---

**Scope reminder:** Orchard's immediate goal is Milestone A, a working nested Wayland compositor with basic floating-window management. Keep the first implementation small and correct. The larger desktop ecosystem can grow around it later.
