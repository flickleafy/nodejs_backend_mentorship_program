# Node.js Backend Mentorship Program

Welcome to the **Node.js Backend Mentorship Program**! This repository is designed to help students and junior developers systematically evolve their backend skills from beginner to senior levels through hands-on, build-style exercises and real-world scenarios.

## Purpose

This program provides a structured, progressive learning path for Node.js backend development. It covers foundational concepts, practical CRUD operations, database persistence, authentication, file handling, integrations, observability, deployment, and more. Each module contains step-by-step rubrics, acceptance criteria, and recommended best practices, making it ideal for self-study, bootcamps, or guided mentorship.

## Structure

The repository is organized into three main folders:

- **junior_level/**: Foundational to intermediate backend skills, with 10 modules:
  1. **Foundations & HTTP**: Learn the basics of Node.js servers, HTTP, and Express.
     1. **API Design & Versioning**: Best practices for RESTful APIs, versioning strategies, and backward compatibility.
     2. **Rate Limiting & Throttling**: Protecting APIs from abuse and ensuring fair usage.
  2. **In-Memory CRUD & Pagination**: Build CRUD APIs and pagination logic using in-memory stores.
  3. **Persistence (SQLite/PostgreSQL, Knex/Sequelize)**: Transition to real databases and ORMs.
     1. **Data Modeling & Validation**: Schema design, normalization, and advanced validation techniques.
  4. **Authentication & Authorization**: Implement secure user registration, login, and access control.
     1. **Advanced Security Fundamentals**: Go beyond the OWASP Top 10 to master secure coding practices, threat modeling, vulnerability scanning, and proactive risk mitigation strategies for modern backend applications.
     2. **Comprehensive Session Management**: Implement robust session handling using cookies, tokens, expiration policies, refresh mechanisms, multi-factor authentication, and advanced techniques to prevent session hijacking, fixation, and other sophisticated attacks.
  5. **Files, Validation, Documentation**: Handle file uploads, data validation, and API documentation.
     1. **Documentation & Developer Experience**: Writing clear API docs, onboarding guides, and code comments.
  6. **Internationalization (i18n) & Localization (l10n)**: Making apps accessible to global users.
  7. **Integrations, Jobs, Caching**: Work with third-party APIs, background jobs, and caching strategies.
  8. **Observability, Testing, Tooling**: Add logging, metrics, tracing, and automated testing.
     1. **Advanced Error Handling & Resilience**: Master robust error management by implementing centralized error handling, designing systems for graceful degradation, and building resilient APIs with retry logic, circuit breakers, and fallback mechanisms to ensure reliability under failure conditions.
     2. **Monitoring & Alerting**: Develop production-grade observability by integrating advanced monitoring tools, setting up automated alerts for critical errors, downtime, and performance anomalies, and leveraging dashboards to proactively detect and respond to issues before they impact users.
  9. **Deployment, Security, Performance**: Containerize apps, orchestrate with Docker Compose, and harden security.
     1. **Advanced Environment Management & Configuration**: Mastering environment variables, hierarchical config systems, secrets rotation, and secure configuration for multi-stage deployments.
     2. **Robust Continuous Integration/Continuous Deployment (CI/CD)**: Designing resilient CI/CD pipelines with automated testing, security scanning (e.g., Snyk), code quality checks (e.g., ESLint, Prettier, SonarQube), performance scans, dependency audits, and integration of tools for monitoring and reporting. Includes strategies for zero-downtime deployments, rollback mechanisms, and orchestrating multi-environment releases.
  10. **Working with Legacy Systems**: Strategies for integrating or refactoring legacy codebases.

- **mid_level/**: Reserved for more advanced backend topics and exercises (to be expanded).
- **senior_level/**: Reserved for senior-level challenges and architectural patterns (to be expanded).

## How to Use

1. **Start at the beginning**: Begin with the first module in `junior_level/1_foundations_http`. Each module builds on the previous. The difficult will increase in small steps, making the transition from one module to another smoother.
2. **Follow the rubrics**: Each module's `readme.md` provides clear steps and acceptance criteria.
3. **Progress at your own pace**: Move to the next module once you meet the acceptance criteria.
4. **Experiment and extend**: Try alternative libraries, add features, or refactor for best practices.
5. **Explore optional advanced introductions**: Subsections, all that are labeled with letters 'a)'... (e.g., `a_api_design_versioning`) are basic introductions to advanced topics. These are optional for the junior level, but completing them will make later steps at mid_level or senior_level much easier and smoother.

## Who Is This For?

- Students and self-learners seeking a practical Node.js backend roadmap.
- Bootcamp participants and instructors.
- Junior developers aiming to reach mid/senior proficiency.
- Mentors looking for a structured curriculum.

## Getting Started

- Clone the repository.
- Pick a module in `junior_level/` and read its `readme.md`.
- Complete the exercises, test your solutions, and seek feedback.

## Contributing

Contributions are welcome! Feel free to submit improvements, new modules, or advanced exercises for mid/senior levels.

## Supporting This Project

If you'd like to support this mentorship program financially, your generosity is truly appreciated! Every little bit helps keep the project growing and accessible for learners everywhere. No pressure—just a heartfelt thank you for considering it. 😊

## 📄 License

This project is licensed under the **GNU General Public License v3.0 (GPL-3.0)** for **personal and non-commercial use only**.

### Personal Use

For personal, educational, and non-commercial purposes, this project is freely available under the GPL-3.0 license:

✅ **You Can**:

- Use these materials for personal projects and learning
- Modify and adapt the materials for non-commercial purposes
- Contribute improvements back to the project

⚠️ **You Must**:

- Disclose source and include license notices
- Share modifications under the same GPL-3.0 license
- Clearly state any significant changes made to original materials

❌ **You Cannot**:

- Sublicense under different terms
- Hold authors liable for damages

### Commercial Use

**Commercial use of this software requires a separate commercial license.**

Commercial use includes, but is not limited to:

- Integration into commercial products or services
- Use within organizations generating revenue
- Deployment in enterprise or production environments for business purposes
- Distribution as part of commercial offerings

For commercial licensing inquiries, please contact inbox.

We offer flexible commercial licensing options tailored to your organization's needs, including support and maintenance agreements.

### Full License Text

The GPL-3.0 license terms for non-commercial use can be found in the [LICENSE](./LICENSE) file.

```text
Copyright (C) 2022-2026 flickleafy

This program is free software for personal use: you can redistribute it 
and/or modify it under the terms of the GNU General Public License as 
published by the Free Software Foundation, either version 3 of the License, 
or (at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

Commercial use requires a separate commercial license. Please contact
the copyright holder for commercial licensing terms.
```

---

**Ready to level up your Node.js backend skills? Start with `junior_level/1_foundations_http` and build your way up!**
