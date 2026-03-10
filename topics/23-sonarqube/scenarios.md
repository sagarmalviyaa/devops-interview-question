# SonarQube — Scenario-Based Questions

---

## S1: The SonarQube Quality Gate is failing on a critical release. The team wants to bypass it. What do you advise?

**Answer:**

**Don't bypass it.** Instead:

1. **Assess the failures:**
   - Are they real bugs or false positives?
   - How severe are the issues?
   - Can they be fixed quickly?

2. **If they're false positives:**
   - Mark them as "Won't Fix" or "False Positive" in SonarQube with justification
   - This won't count against the Quality Gate

3. **If they're real but non-critical:**
   - Fix what you can quickly
   - For the rest, create tickets and fix in the next sprint
   - Temporarily adjust the Quality Gate threshold (with team agreement and a plan to restore it)

4. **If bypassing is truly necessary (emergency):**
   - Document the decision and who approved it
   - Create a follow-up ticket with a deadline
   - Review the issues in the next sprint
   - Never make bypassing a habit

**Key message:** Quality Gates exist to prevent problems in production. Bypassing them should be extremely rare and always documented.

---

## S2: Your team's code coverage is at 30% and the Quality Gate requires 80% on new code. How do you improve?

**Answer:**

1. **Focus on new code first** ("Clean as You Code"):
   - SonarQube's default gate checks coverage on NEW code only
   - Don't try to retroactively cover all old code

2. **Set realistic targets:**
   - Start with 60% on new code, gradually increase to 80%
   - Focus on critical paths first (payment, authentication)

3. **Make testing part of the workflow:**
   - Require tests in pull requests
   - Use pre-commit hooks to run tests locally
   - Pair testing with code review

4. **Provide testing resources:**
   - Training on testing frameworks
   - Testing templates and examples
   - Dedicated time for writing tests

5. **Track progress:**
   - Monitor coverage trends in SonarQube dashboards
   - Celebrate improvements
   - Identify and address areas with consistently low coverage