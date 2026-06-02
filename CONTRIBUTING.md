# Contributing to Ember.js Agent Skills

Thank you for contributing to the Skill.js Ember.js collection! To ensure these skills remain highly effective for AI coding agents, please follow these guidelines based on the [AgentSkills.io Best Practices](https://agentskills.io/skill-creation/best-practices).

## 1. Follow the Specification
- Every skill must live in its own directory with a `SKILL.md` file.
- The `SKILL.md` must include valid YAML frontmatter:
  ```yaml
  ---
  name: ember-skill-name
  description: Clear description of what the skill does and when to use it (max 1024 chars).
  ---
  ```
- The `name` must match the directory name exactly (lowercase, hyphens only, max 64 chars).

## 2. Differentiate Modern vs Legacy
Ember has a long history. AI agents frequently hallucinate legacy patterns. Your skill must **explicitly teach the modern pattern** while **warning against the legacy pattern**.
- Do teach: `@tracked`, `@glimmer/component`, `.gjs`/`.gts`, `ember-resources`.
- Do explicitly warn against: `Ember.Object.extend`, `this.get()`, Mixins, `@computed`.

## 3. Use Progressive Disclosure
- Keep `SKILL.md` concise (under 500 lines).
- Provide immediate, high-value guidance and code templates.
- If a skill requires extensive reference material (like full API docs), place those in a `references/` directory and instruct the agent when to read them.

## 4. Include "Gotchas"
The most valuable content for an AI is knowing what *not* to do. Include a **Gotchas** section in your `SKILL.md` that highlights specific Ember quirks the AI is likely to get wrong (e.g., component arguments being read-only, avoiding DOM manipulation in lifecycle hooks).

## 5. Be Specific
Don't say "Write good tests." Do say "Use `setupRenderingTest(hooks)` and `await render(...)`." Provide exact import paths and exact code snippets.

## Validating Your Skill
If you have `skills-ref` installed, run:
```bash
skills-ref validate ./skills/your-skill-name
```
