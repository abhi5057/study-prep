# Lead Experience Engineer Interview Prep (Based on XT Playbook + Assumed Role Requirements)

Assumption note:
- `XT Deck (1)` and `Lead Experience Engineer` PDF are not present in this workspace.
- This prep is built from the closest matching source: `26_NEXTJS_ARCHITECT_PRODUCTION_PLAYBOOK.md` plus standard Lead Experience Engineer expectations.
- When you share the exact deck/PDF, this can be aligned line-by-line in minutes.

## 1) Link Check From XT Source (Current Workspace)

Observed in `26_NEXTJS_ARCHITECT_PRODUCTION_PLAYBOOK.md`:
- Links found are mostly sample endpoints (`https://api.example.com/...`) and local telemetry endpoint (`http://localhost:4317`).
- These are example code links, not official study resources.

Conclusion:
- For thorough preparation, use the curated official resources in Section 8.

## 2) What Lead Experience Engineer Interviews Usually Test

1. Product + UX engineering judgment
- Can you connect technical choices to user outcomes?

2. Frontend architecture depth
- Rendering strategy, state boundaries, data contracts, and performance.

3. Reliability and operability
- Failure modes, graceful degradation, observability, release safety.

4. Leadership and execution
- Mentorship, standards, cross-team decisions, incident ownership.

5. Communication quality
- Structured reasoning under ambiguity.

Use this response frame:
- Context -> User impact -> Constraints -> Options -> Decision -> Risks -> Metrics.

## 3) High-Yield Topics For Tomorrow (Priority Order)

1. Next.js rendering and caching strategy (App Router, RSC boundaries).
2. Core Web Vitals and performance remediation plan.
3. API/data flow design and failure handling.
4. Security and privacy-by-default in frontend/backend boundaries.
5. Deployment safety: canary, rollback, error budget guardrails.
6. Team-level engineering practices and decision governance.

## 4) Must-Have Architecture Narratives (Prepare Verbatim)

### Story A: Performance Recovery

Prompt: "Your web app regressed after migration."

Answer skeleton:
- Symptoms: CWV regression (LCP/INP/CLS), bounce increase.
- Diagnosis: route-level bundle inflation, hydration work, cache misses.
- Fixes: split server/client boundaries, prefetch discipline, image/font optimization, selective ISR.
- Guardrail: CWV budgets in CI + dashboard SLO alerts.
- Outcome: measurable p75/p95 improvement.

### Story B: Reliability Under Dependency Failure

Prompt: "Third-party API is intermittently failing."

Answer skeleton:
- Add timeout budget and fallback UX states.
- Use stale-while-revalidate where correctness allows.
- Apply retry with jitter only for safe idempotent operations.
- Expose dependency health in telemetry and release gates.

### Story C: Platform Standardization Across Teams

Prompt: "How do you align 10+ teams?"

Answer skeleton:
- Standard template + lint/security/perf checks.
- Shared observability and error taxonomy.
- Controlled exceptions process with architecture review.
- Quarterly pruning of framework/platform debt.

## 5) L5 Grilling Questions You Should Practice

1. How do you decide SSR vs SSG vs ISR vs client fetch for a route?
2. How do you prevent cache invalidation storms after CMS updates?
3. Why can RSC misuse increase latency even if JS bundle shrinks?
4. How do you prove an optimization improved user-perceived performance?
5. What architecture guardrails do you enforce org-wide and why?
6. How do you balance design-system consistency with team velocity?
7. How do you run incident command for frontend-major outages?
8. What decision did you reverse recently and what signal triggered it?
9. How do you evaluate edge runtime suitability for a feature?
10. How do you align product priorities with engineering quality constraints?

## 6) Coding/System Design Rounds (Likely)

### Round 1: Design a global content page

Expectations:
- Multi-region caching strategy.
- Localization strategy.
- Fallback behavior during CMS outage.
- Metrics and rollback plan.

### Round 2: Build resilient checkout-like flow

Expectations:
- Idempotent API contract.
- Partial failure UX states.
- Telemetry for conversion-impacting errors.
- Retries only where safe.

### Round 3: Debug production incident

Expectations:
- Structured triage sequence.
- Clear blast radius and mitigation.
- Root cause and preventive controls.

## 7) 90-Minute Final Revision Plan

0-20 min:
- Rehearse rendering/caching decision matrix with one real project example.

20-40 min:
- Rehearse performance narrative with metrics (before/after).

40-60 min:
- Rehearse reliability/failure-mode narrative and incident command flow.

60-75 min:
- Rehearse 10 grilling questions from Section 5.

75-90 min:
- Final pass on intro and closing statements.

## 8) Thorough Official Resource Material (Curated)

Core framework:
- Next.js docs: https://nextjs.org/docs
- React docs: https://react.dev
- TypeScript docs: https://www.typescriptlang.org/docs/

Performance and UX:
- Web Vitals: https://web.dev/vitals/
- MDN performance guide: https://developer.mozilla.org/en-US/docs/Web/Performance
- Lighthouse docs: https://developer.chrome.com/docs/lighthouse

Accessibility and inclusive design:
- WCAG overview: https://www.w3.org/WAI/standards-guidelines/wcag/
- ARIA authoring practices: https://www.w3.org/WAI/ARIA/apg/

Reliability and operations:
- OpenTelemetry docs: https://opentelemetry.io/docs/
- SRE book (Google): https://sre.google/sre-book/table-of-contents/

Testing and quality:
- Playwright: https://playwright.dev/docs/intro
- Jest: https://jestjs.io/docs/getting-started
- Testing Library: https://testing-library.com/docs/

Security:
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/

## 9) Interview-Day Script (Use As-Is)

Opening:
- "I optimize for user experience outcomes with measurable guardrails: performance, reliability, and delivery safety."

When asked for design choice:
- "Given these constraints, I considered A and B. I chose B because it improves user latency and operational safety, with these trade-offs and controls."

When challenged:
- "Great point. The risk is real; I would mitigate with phased rollout, telemetry thresholds, and rollback criteria defined before launch."

Closing:
- "I bring architecture depth plus execution discipline: clear standards, measurable outcomes, and strong incident ownership."

## 10) What To Avoid

- Generic framework talk without user/business impact.
- Performance claims without metric evidence.
- Resilience claims without failure-mode details.
- Leadership claims without concrete team/process outcomes.

## 11) Fast Follow (When PDF/Deck Is Shared)

I can immediately produce:
- Requirement-to-answer mapping table.
- Top 25 company-specific probable questions.
- Tailored STAR stories aligned to the exact PDF language.