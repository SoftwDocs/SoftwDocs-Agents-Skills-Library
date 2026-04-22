# 🚀 Claude Code Skills Library

[![Skills Count](https://img.shields.io/badge/skills-9-blue?style=flat-square)](./#-available-skills)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](./LICENSE)
[![Validation](https://img.shields.io/badge/validation-passing-brightgreen?style=flat-square)](./.github/workflows/validate-skills.yml)
[![Contributors Welcome](https://img.shields.io/badge/contributors-welcome-orange?style=flat-square)](./CONTRIBUTING.md)

> **Enterprise-grade instruction manuals for AI coding agents**

Welcome to the ultimate repository of **Claude Code Skills** — a curated collection of high-performance, reusable instruction manuals (markdown-based skills) designed to make AI coding agents smarter, faster, and more consistent.

**Vision:** Building the foundation for a premium marketplace like *skills.sh* where developers can access elite AI workflows.

---

## 📦 Repository Stats

| Metric | Value |
|--------|-------|
| **Total Skills** | 9 |
| **Total Lines** | ~7,000+ |
| **Technologies** | Next.js, FastAPI, TypeScript, Prisma, Tailwind |
| **License** | MIT |
| **Status** | Production Ready |

---

## 🛠️ How to Use
To use these skills in your project, simply copy the desired skill folder into your local project's directory:
`your-project/.claude/skills/`

---

## 🌟 Available Skills

### 1. [Agentic Workflow Orchestrator](./agentic-workflow-orchestrator/SKILL.md)
**Purpose:** Advanced multi-agent task orchestration with dependency management, state tracking, and validation gates. Transforms Claude into a sophisticated project manager capable of handling complex, multi-phase development workflows.
- **Best for:** Large-scale feature development, autonomous agent systems, and complex project decomposition.
- **Features:** Task maps, sub-agent delegation, context preservation, error recovery, and parallel execution patterns.

### 2. [Performance-First Next.js](./performance-first-nextjs/SKILL.md)
**Purpose:** Comprehensive optimization guide for achieving 95+ Lighthouse scores in Next.js 14+ applications. Covers Core Web Vitals, image optimization, caching strategies, and bundle optimization.
- **Best for:** E-commerce platforms, high-traffic SaaS applications, and SEO-critical websites.
- **Features:** LCP optimization, dynamic imports, server components, streaming, ISR caching, and edge runtime.

### 3. [Enterprise CRUD Engine](./enterprise-crud-engine/SKILL.md)
**Purpose:** Production-grade CRUD implementation with TanStack Query, Shadcn/UI DataTables, optimistic updates, and complete audit trails. Built for enterprise data-heavy applications.
- **Best for:** ERP systems, admin dashboards, inventory management, and data-intensive applications.
- **Features:** Server Actions with auth, optimistic updates, pagination, filtering, soft deletes, and compliance logging.

### 4. [Tailwind Glass UI](./tailwind-glass-ui/SKILL.md)
**Purpose:** Premium glassmorphism (frosted glass) design system for modern, high-end interfaces. Advanced translucent effects with backdrop filters and sophisticated animations.
- **Best for:** Premium SaaS landing pages, dashboards, portfolio sites, and modern web applications.
- **Features:** Glass cards, animated backgrounds, gradient effects, hover interactions, and responsive layouts.

### 5. [API Security Shield](./api-security-shield/SKILL.md)
**Purpose:** Enterprise-grade API security implementation for Next.js and FastAPI. Defense-in-depth strategy covering authentication, authorization, and OWASP compliance.
- **Best for:** Production APIs, healthcare applications, fintech systems, and any security-critical backend.
- **Features:** JWT with RS256, rate limiting, CORS, input validation, SQL injection prevention, XSS protection, CSRF tokens, and security headers.

### 6. [Auto-Test Architect](./auto-test-architect/SKILL.md)
**Purpose:** Comprehensive testing strategy covering unit, integration, and E2E testing with automated CI/CD pipelines. Ensures code quality and reliability at every layer.
- **Best for:** Production applications, team collaborations, and projects requiring high reliability.
- **Features:** Vitest unit tests, Playwright E2E, visual regression, MSW mocking, coverage thresholds, and GitHub Actions integration.

### 7. [Structured AI Output](./structured-ai-output/SKILL.md)
**Purpose:** Force LLMs to return strict, validated JSON output using Zod schemas. Essential for reliable agentic workflows and AI automation.
- **Best for:** AI agents, automated data extraction, code generation workflows, and LLM-powered applications.
- **Features:** Multi-provider support (OpenAI, Anthropic, Gemini), retry logic, streaming, and type-safe responses.

### 8. [SEO Master](./seo-master/SKILL.md)
**Purpose:** Enterprise SEO automation for Next.js applications. Dynamic metadata, OpenGraph images, structured data, and search ranking optimization.
- **Best for:** Content websites, e-commerce stores, blogs, and marketing sites requiring organic traffic.
- **Features:** Dynamic OG images, JSON-LD structured data, sitemap generation, robots.txt, and Core Web Vitals for SEO.

### 9. [Pharmacy POS Logic](./pharmacy-pos-logic/SKILL.md)
**Purpose:** Specialized business logic for pharmacy and medical store management systems. Prescription validation, drug interactions, and regulatory compliance.
- **Best for:** Pharmacy management systems, medical store POS, healthcare inventory, and prescription processing.
- **Features:** Prescription validation, drug interaction checking, FIFO inventory, expiry tracking, insurance claims, and HIPAA compliance.

---

## 📊 Skills Quick Reference

| Category | Skills |
|----------|--------|
| **Frontend** | Performance-First Next.js, Tailwind Glass UI, SEO Master |
| **Backend** | Enterprise CRUD Engine, API Security Shield |
| **AI/ML** | Agentic Workflow Orchestrator, Structured AI Output |
| **Testing** | Auto-Test Architect |
| **Domain-Specific** | Pharmacy POS Logic |

---

## ⏳ Coming Soon (Premium Skills)

We are actively developing the following advanced skills:

- 🎨 **Advanced Animation Engine:** GSAP + Framer Motion complex animations
- 🔒 **HIPAA Compliance Kit:** Healthcare data handling and audit systems
- 📱 **React Native Architect:** Cross-platform mobile app development
- 🐳 **DevOps Pipeline:** Docker, Kubernetes, and CI/CD automation
- 🤝 **Third-Party Integrations:** Stripe, Twilio, SendGrid implementation patterns

---

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details on:
- Creating new skills
- Improving existing skills
- Submitting pull requests
- Code of conduct

### Quick Contribution Steps
1. 🍴 Fork the repository
2. 🌿 Create your feature branch: `git checkout -b skill/amazing-skill`
3. ✍️ Add your skill following our [template](./CONTRIBUTING.md#skill-structure)
4. ✅ Ensure validation passes: [Workflow Guide](./.github/workflows/validate-skills.yml)
5. � Submit a pull request using our [template](./.github/pull_request_template.md)

---

## 🐛 Issues & Support

- **Bug Reports**: [Use template](./.github/ISSUE_TEMPLATE/bug_report.md)
- **Skill Requests**: [Use template](./.github/ISSUE_TEMPLATE/skill_request.md)
- **Improvements**: [Use template](./.github/ISSUE_TEMPLATE/skill_improvement.md)

---

## � About the Author

<table>
<tr>
<td>
<img src="https://avatars.githubusercontent.com/syed-mujtaba-stack" width="100" height="100" style="border-radius: 50%">
</td>
<td>
<strong>Syed Mujtaba Abbas</strong><br>
Co-founder & Lead Developer at <strong>SoftwDocs</strong><br>
Full Stack & Agentic AI Developer<br>
<a href="https://mujtaba-abbas.web.app">Portfolio</a> • 
<a href="https://github.com/syed-mujtaba-stack">GitHub</a> • 
<a href="https://linkedin.com/in/mujtabaabbas">LinkedIn</a>
</td>
</tr>
</table>

**Mission:** Building the future of AI-driven software solutions and democratizing elite development workflows.

---

## 📊 Repository Activity

![GitHub Contributors](https://img.shields.io/github/contributors/etictech/skills-liabrary?style=flat-square)
![GitHub Last Commit](https://img.shields.io/github/last-commit/etictech/skills-liabrary?style=flat-square)
![GitHub Repo Size](https://img.shields.io/github/repo-size/etictech/skills-liabrary?style=flat-square)

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

The skills content (instruction manuals, patterns, best practices) are provided under MIT terms with additional attribution guidelines for public distribution.

---

<div align="center">

**⭐ Star this repository if you find it useful!**

Built with ❤️ by <a href="https://softwdocs.com">SoftwDocs</a> Team

</div>
