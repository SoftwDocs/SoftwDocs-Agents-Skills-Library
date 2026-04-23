# 🚀 Claude Code Skills Library

[![Skills Count](https://img.shields.io/badge/skills-19-blue?style=flat-square)](./#-available-skills)
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
| **Total Skills** | 19 |
| **Total Lines** | ~15,000+ |
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

### 10. [Advanced Animation Engine](./advanced-animation-engine/SKILL.md)
**Purpose:** Professional animation system using GSAP and Framer Motion for complex, performant web animations. Includes timeline orchestration, scroll-based animations, physics simulations, and micro-interactions.
- **Best for:** Interactive landing pages, scroll-triggered storytelling, physics-based UI interactions, and animated data visualizations.
- **Features:** GSAP timelines, ScrollTrigger, Framer Motion gestures, physics simulations, and performance optimization.

### 11. [HIPAA Compliance Kit](./hipaa-compliance-kit/SKILL.md)
**Purpose:** Complete HIPAA compliance implementation for healthcare applications. Covers PHI encryption, audit logging, access controls, BAA templates, and regulatory requirements handling.
- **Best for:** Healthcare applications, telemedicine platforms, EHR systems, medical device software, and pharmacy management.
- **Features:** PHI encryption, audit logging, RBAC, minimum necessary principle, BAA templates, and incident response.

### 12. [React Native Architect](./react-native-architect/SKILL.md)
**Purpose:** Complete React Native architecture for cross-platform mobile development. Covers navigation, state management, native modules, performance optimization, and platform-specific implementations.
- **Best for:** Cross-platform mobile applications, web-to-mobile migration, native-feeling mobile experiences, and apps with offline capabilities.
- **Features:** React Navigation, Redux Toolkit, React Query, native modules, Hermes engine, and performance optimization.

### 13. [DevOps Pipeline](./devops-pipeline/SKILL.md)
**Purpose:** Complete DevOps pipeline implementation with Docker, Kubernetes, CI/CD automation, infrastructure as code, monitoring, and deployment strategies for production applications.
- **Best for:** Production deployment pipelines, container-based infrastructure, CI/CD automation, multi-environment deployments, and Kubernetes orchestration.
- **Features:** Docker multi-stage builds, Kubernetes manifests, GitHub Actions, Terraform, Prometheus/Grafana, and deployment strategies.

### 14. [Third-Party Integrations](./third-party-integrations/SKILL.md)
**Purpose:** Complete implementation patterns for third-party service integrations including Stripe payments, Twilio communications, SendGrid emails, AWS services, and OAuth authentication with proper error handling and security.
- **Best for:** Payment processing, SMS/voice communications, transactional emails, cloud services integration, and social authentication.
- **Features:** Stripe payments, Twilio SMS/voice, SendGrid emails, AWS S3/SNS/SES, OAuth 2.0, and webhook security.

### 15. [GraphQL API Architect](./graphql-api-architect/SKILL.md)
**Purpose:** Complete GraphQL API architecture with Apollo Server, schema design, resolvers, subscriptions, authentication, caching, and best practices for production GraphQL APIs.
- **Best for:** Building GraphQL APIs, migrating REST to GraphQL, real-time features, and complex schema design.
- **Features:** Apollo Server, type-safe schemas, DataLoader, subscriptions, caching, and error handling.

### 16. [Real-Time Communication](./real-time-communication/SKILL.md)
**Purpose:** Complete real-time communication implementation with WebSockets, Socket.IO, WebRTC, push notifications, and live collaboration features for modern web applications.
- **Best for:** Chat applications, live collaboration, video/audio calling, real-time dashboards, and multiplayer games.
- **Features:** Socket.IO, WebRTC, push notifications, live editing, and WebSocket security.

### 17. [Serverless Architecture](./serverless-architecture/SKILL.md)
**Purpose:** Complete serverless architecture implementation with AWS Lambda, API Gateway, DynamoDB, S3, CloudFormation, and best practices for scalable, cost-effective serverless applications.
- **Best for:** Serverless APIs, event-driven architectures, cost-effective applications, and microservices without servers.
- **Features:** AWS Lambda, API Gateway, DynamoDB, SAM, CloudFormation, and event triggers.

### 18. [Caching Strategies](./caching-strategies/SKILL.md)
**Purpose:** Complete caching strategies implementation with Redis, Memcached, CDN, browser caching, database query caching, and cache invalidation patterns for high-performance applications.
- **Best for:** Reducing database load, improving API response times, session storage, and performance optimization.
- **Features:** Redis, multi-level caching, CDN integration, cache invalidation, and monitoring.

### 19. [Event-Driven Architecture](./event-driven-architecture/SKILL.md)
**Purpose:** Complete event-driven architecture implementation with message queues, event buses, pub/sub patterns, saga patterns, and distributed systems for scalable, decoupled applications.
- **Best for:** Microservices architectures, asynchronous processing, distributed transactions, and real-time data pipelines.
- **Features:** RabbitMQ, SQS, Kafka, saga pattern, event sourcing, and CQRS.

---

## 📊 Skills Quick Reference

| Category | Skills |
|----------|--------|
| **Frontend** | Performance-First Next.js, Tailwind Glass UI, SEO Master, Advanced Animation Engine |
| **Backend** | Enterprise CRUD Engine, API Security Shield, Third-Party Integrations, GraphQL API Architect |
| **AI/ML** | Agentic Workflow Orchestrator, Structured AI Output |
| **Mobile** | React Native Architect |
| **DevOps** | DevOps Pipeline, Serverless Architecture |
| **Real-Time** | Real-Time Communication |
| **Performance** | Caching Strategies |
| **Architecture** | Event-Driven Architecture |
| **Testing** | Auto-Test Architect |
| **Compliance** | HIPAA Compliance Kit |
| **Domain-Specific** | Pharmacy POS Logic |

---

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

![GitHub Contributors](https://img.shields.io/github/contributors/syed-mujtaba-stack/skills-liabrary?style=flat-square)
![GitHub Last Commit](https://img.shields.io/github/last-commit/syed-mujtaba-stack/skills-liabrary?style=flat-square)
![GitHub Repo Size](https://img.shields.io/github/repo-size/syed-mujtaba-stack/skills-liabrary?style=flat-square)

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

The skills content (instruction manuals, patterns, best practices) are provided under MIT terms with additional attribution guidelines for public distribution.

---

<div align="center">

**⭐ Star this repository if you find it useful!**

Built with ❤️ by <a href="https://softwdocs.com">SoftwDocs</a> Team

</div>
