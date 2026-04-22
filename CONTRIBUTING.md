# 🤝 Contributing to Claude Code Skills Library

Thank you for your interest in contributing! This guide will help you create high-quality skills that benefit the entire community.

---

## 📋 Table of Contents

- [Getting Started](#getting-started)
- [Skill Structure](#skill-structure)
- [Writing Guidelines](#writing-guidelines)
- [Submission Process](#submission-process)
- [Review Criteria](#review-criteria)

---

## 🚀 Getting Started

### Prerequisites
- Familiarity with the technology you're documenting
- Understanding of best practices and common patterns
- Ability to write clear, technical documentation

### Setup
1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your skill: `git checkout -b skill/your-skill-name`

---

## 📁 Skill Structure

Each skill must follow this directory structure:

```
skills-liabrary/
├── your-skill-name/
│   └── SKILL.md          # Main skill file (required)
├── another-skill/
│   └── SKILL.md
└── README.md             # Central index
```

### SKILL.md Template

```markdown
---
name: your-skill-name
description: Clear, concise description of what this skill does
tags: [tag1, tag2, tag3, relevant, keywords]
version: 1.0.0
author: Your Name
---

# Skill Title

## Overview
Brief explanation of the skill's purpose and value.

## When to Use This Skill
- Scenario 1
- Scenario 2
- Scenario 3

## Core Concepts

### Concept 1
Detailed explanation with code examples.

```typescript
// TypeScript example
const example = "production-ready code";
```

### Concept 2
More detailed content...

## Implementation Patterns

### Pattern 1: Basic Usage
Step-by-step implementation guide.

### Pattern 2: Advanced Usage
Complex scenarios and edge cases.

## Best Practices
- ✅ Do this
- ❌ Don't do this
- 💡 Pro tip

## Common Pitfalls
1. **Mistake 1**: Explanation and solution
2. **Mistake 2**: Explanation and solution

## Integration Examples

### With Other Skills
How this skill works with others in the library.

### Technology Stack
Compatible frameworks and tools.

## Testing & Validation
How to verify implementations using this skill.

## Resources
- [Link 1](url) - Description
- [Link 2](url) - Description
```

---

## ✍️ Writing Guidelines

### Content Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Total Lines | 100 | 200+ |
| Code Examples | 3 | 5+ |
| Sections | 5 | 8+ |

### Quality Standards

1. **Clarity First**
   - Use clear, concise language
   - Avoid unnecessary jargon
   - Explain acronyms on first use

2. **Code Quality**
   - All code must be TypeScript
   - Use strict typing (no `any`)
   - Include error handling
   - Follow modern best practices

3. **Practical Focus**
   - Real-world examples
   - Production-ready patterns
   - Edge case handling
   - Performance considerations

4. **Complete Coverage**
   - Setup instructions
   - Implementation steps
   - Testing guidance
   - Troubleshooting tips

### Style Guide

- Use `##` for main sections
- Use `###` for subsections
- Use backticks for code, filenames, and technical terms
- Use emojis sparingly for visual organization
- Include a table of contents for skills >150 lines

---

## 📤 Submission Process

### 1. Create Your Skill
Follow the structure and guidelines above.

### 2. Self-Review Checklist
Before submitting, verify:
- [ ] Minimum 100 lines of content
- [ ] Proper frontmatter with all required fields
- [ ] At least 3 code examples
- [ ] No placeholder or TODO content
- [ ] All code is valid TypeScript
- [ ] Links are functional
- [ ] README.md updated with your skill

### 3. Submit Pull Request
1. Push your branch to your fork
2. Open a PR against the main repository
3. Fill out the PR template completely
4. Wait for review and feedback

---

## 🔍 Review Criteria

All submissions are evaluated on:

| Criteria | Weight | Description |
|----------|--------|-------------|
| **Completeness** | 25% | Covers topic thoroughly |
| **Code Quality** | 25% | Clean, modern, type-safe |
| **Practical Value** | 20% | Solves real problems |
| **Documentation** | 15% | Clear and well-organized |
| **Originality** | 15% | Unique approach or insight |

### Review Timeline
- Initial review: 2-3 business days
- Feedback incorporation: 1-2 iterations
- Final merge: After approval

---

## 🏷️ Skill Categories

When proposing a new skill, categorize it appropriately:

- **Frontend**: UI/UX, React, Next.js, CSS, Animation
- **Backend**: APIs, Databases, Server, Authentication
- **AI/ML**: LLMs, Agents, Machine Learning
- **Testing**: Unit, Integration, E2E, QA
- **DevOps**: CI/CD, Docker, Cloud, Monitoring
- **Security**: Auth, Encryption, Compliance
- **Domain-Specific**: Industry-specific (healthcare, finance, etc.)

---

## 💡 Tips for Success

1. **Study Existing Skills**: Review current skills for patterns and quality standards
2. **Focus on One Thing**: Each skill should excel at one specific purpose
3. **Test Your Examples**: Ensure all code compiles and works
4. **Be Detail-Oriented**: Small details make a big difference
5. **Stay Current**: Use latest stable versions of technologies

---

## 📞 Questions?

- Open an issue for discussion before major work
- Join community discussions in existing issues
- Tag maintainers for clarification

---

## 🙏 Thank You!

Your contributions help build the premier AI coding skills library. Every skill added makes AI agents more capable and developers more productive.

**Happy contributing! 🚀**
