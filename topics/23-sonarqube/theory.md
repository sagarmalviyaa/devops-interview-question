# SonarQube — Theory Questions

---

## Q1: What is SonarQube?

**Answer:**
**SonarQube** is an open-source platform for continuous code quality inspection. It automatically analyzes code to find bugs, vulnerabilities, code smells (bad patterns), and technical debt.

**What it detects:**
- **Bugs** — Code that will likely cause errors at runtime
- **Vulnerabilities** — Security weaknesses that could be exploited
- **Code Smells** — Maintainability issues (not bugs, but bad practices)
- **Duplications** — Copy-pasted code
- **Coverage** — How much code is covered by tests

---

## Q2: What is a Quality Gate?

**Answer:**
A **Quality Gate** is a set of conditions that code must meet before it can pass. If any condition fails, the gate fails, and the code shouldn't be merged/deployed.

**Default Quality Gate conditions:**
- No new bugs
- No new vulnerabilities
- Code coverage on new code ≥ 80%
- Duplicated lines on new code < 3%
- Maintainability rating is A

```
Developer pushes code → SonarQube analyzes → Quality Gate check
  ├── Pass ✅ → Merge allowed
  └── Fail ❌ → Merge blocked, developer must fix issues
```

---

## Q3: How do you integrate SonarQube with CI/CD?

**Answer:**

**GitHub Actions:**
```yaml
- name: SonarQube Scan
  uses: SonarSource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

- name: Quality Gate Check
  uses: SonarSource/sonarqube-quality-gate-action@master
  timeout-minutes: 5
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Project configuration (sonar-project.properties):**
```properties
sonar.projectKey=myapp
sonar.sources=src
sonar.tests=tests
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.exclusions=**/node_modules/**,**/dist/**
```

---

## Q4: What is technical debt in SonarQube?

**Answer:**
**Technical debt** is the estimated time it would take to fix all code quality issues. SonarQube calculates it based on the number and severity of issues.

**Example:** "This project has 5 days of technical debt" means it would take approximately 5 developer-days to fix all code smells and maintainability issues.

**Ratings:**
- **A** — Technical debt ratio ≤ 5% (excellent)
- **B** — 6–10%
- **C** — 11–20%
- **D** — 21–50%
- **E** — > 50% (critical)

**Best practice:** Focus on keeping new code clean ("Clean as You Code" approach) rather than trying to fix all existing issues at once.