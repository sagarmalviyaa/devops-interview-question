# Behavioral & Situational Interview Questions

These questions assess your soft skills, problem-solving approach, and real-world experience. Use the **STAR method** (Situation, Task, Action, Result) to structure your answers.

---

## Q1: Tell me about a time when a production system went down. How did you handle it?

**Sample Answer Structure:**

**Situation:** "Our e-commerce platform went down during Black Friday due to a database connection pool exhaustion."

**Task:** "As the on-call engineer, I needed to restore service as quickly as possible while minimizing data loss."

**Action:**
- "I immediately checked our monitoring dashboards (Grafana) and identified the database as the bottleneck"
- "I increased the connection pool limits as a temporary fix"
- "I communicated status updates to stakeholders every 15 minutes"
- "After restoring service, I led the post-mortem"

**Result:**
- "Service was restored in 23 minutes"
- "We implemented connection pooling with PgBouncer to prevent recurrence"
- "We added alerts for connection pool usage"
- "Documented the incident and shared learnings with the team"

---

## Q2: How do you handle a situation where a developer wants to deploy on Friday afternoon?

**Answer:**

"I'd approach this diplomatically but firmly:

1. **Understand the urgency** — Is it a critical hotfix or a feature that can wait?
2. **If it's a hotfix:** Deploy it, but with extra precautions:
   - Ensure someone is available to monitor over the weekend
   - Have a rollback plan ready
   - Keep the change as small as possible
3. **If it's a feature:** Suggest deploying Monday morning:
   - 'I understand you want to ship this, but if something goes wrong, we won't have full team support over the weekend'
   - 'Let's deploy first thing Monday when we have the whole team available'
4. **Long-term:** Establish a deployment policy:
   - No non-critical deployments after Thursday 3 PM
   - Exceptions require on-call coverage commitment"

---

## Q3: Tell me about a time you automated something that saved significant time.

**Sample Answer:**

"At my previous company, developers spent 2 hours per week manually deploying to staging environments.

**What I did:**
- Built a CI/CD pipeline with GitHub Actions
- Automated testing, building, and deploying to staging on every PR merge
- Added Slack notifications for deployment status
- Created a self-service portal for developers to trigger deployments

**Result:**
- Deployment time went from 2 hours to 5 minutes
- Developers could deploy 10x more frequently
- Fewer deployment errors (human error eliminated)
- Saved approximately 100 developer-hours per month across the team"

---

## Q4: How do you prioritize when multiple teams need your help simultaneously?

**Answer:**

"I prioritize based on impact and urgency:

1. **Production issues** — Always first. If production is down, everything else waits
2. **Blocking issues** — If a team is completely blocked and can't work
3. **Upcoming deadlines** — Help teams with imminent deadlines
4. **Improvements** — Non-urgent optimizations and enhancements

**My approach:**
- Acknowledge all requests quickly (even if I can't help immediately)
- Set clear expectations: 'I'll help you after I resolve the production issue, estimated 2 hours'
- Document common requests and create self-service solutions
- If I'm consistently overloaded, raise it with management to get more resources"

---

## Q5: How do you stay current with DevOps technologies?

**Answer:**

"I use a combination of approaches:

- **Daily:** Read DevOps newsletters (DevOps Weekly, TLDR), follow key people on Twitter/LinkedIn
- **Weekly:** Listen to podcasts (Ship It!, The Changelog), read blog posts
- **Monthly:** Try new tools in a home lab or sandbox environment
- **Quarterly:** Take an online course or get a certification
- **Community:** Attend local meetups, participate in online communities (Reddit, Discord)
- **At work:** Propose proof-of-concepts for new tools, share learnings in team meetings

I focus on understanding concepts rather than memorizing tools, because tools change but principles remain."

---

## Q6: Tell me about a time you disagreed with a team member about a technical decision.

**Sample Answer:**

"A colleague wanted to use a complex microservices architecture for a small internal tool. I believed a monolith would be simpler and faster to build.

**How I handled it:**
1. I listened to their reasoning (scalability concerns, technology learning)
2. I presented my perspective with data (team size, expected load, timeline)
3. We agreed to evaluate both approaches against our requirements
4. We created a simple decision matrix (complexity, time to build, maintenance)
5. We agreed on a modular monolith — simple to start, easy to split later if needed

**Key:** I focused on the problem, not the person. We used data to make the decision, not opinions."

---

## Q7: How do you handle on-call responsibilities?

**Answer:**

"I take on-call seriously because it directly impacts users:

**Preparation:**
- Ensure runbooks are up-to-date for common issues
- Test alerting channels (PagerDuty, phone, Slack)
- Review recent changes that might cause issues

**During on-call:**
- Respond to alerts within 5 minutes
- Follow the incident response process (acknowledge → investigate → mitigate → resolve)
- Communicate status to stakeholders
- Document everything for the post-mortem

**After incidents:**
- Write a blameless post-mortem
- Identify action items to prevent recurrence
- Share learnings with the team

**Improving on-call:**
- Reduce alert noise (only alert on actionable issues)
- Automate common fixes (self-healing)
- Rotate on-call fairly across the team
- Track on-call metrics (MTTA, MTTR, alert volume)"

---

## Q8: Describe your approach to writing documentation.

**Answer:**

"I believe in documentation that people actually read and use:

1. **README.md** — Every project gets one: what it does, how to set it up, how to deploy
2. **Runbooks** — Step-by-step guides for operational tasks and incident response
3. **Architecture Decision Records (ADRs)** — Document why decisions were made
4. **Diagrams** — Architecture diagrams (I use Mermaid or draw.io)
5. **Inline comments** — For complex code, explain the 'why' not the 'what'

**My rules:**
- Write docs as you build (not after)
- Keep docs close to the code (in the same repo)
- Review docs in PRs (just like code)
- If someone asks the same question twice, write it down
- Delete outdated docs (wrong docs are worse than no docs)"

---

## Q9: How do you approach learning a new tool or technology?

**Answer:**

"I follow a structured approach:

1. **Understand the 'why'** — What problem does this tool solve? Why was it created?
2. **Quick start** — Follow the official getting started guide
3. **Build something small** — A proof-of-concept or toy project
4. **Read the docs** — Focus on concepts, architecture, and best practices
5. **Apply it** — Use it in a real (non-critical) project
6. **Teach it** — Write a blog post or give a team presentation
7. **Go deeper** — Read source code, understand internals, contribute

I find that teaching forces me to truly understand something."

---

## Q10: What's the most challenging DevOps problem you've solved?

**Tips for answering:**
- Choose a real, specific example
- Show your problem-solving process
- Highlight collaboration with others
- Quantify the impact
- Mention what you learned

**Structure:**
1. What was the problem? (Be specific)
2. Why was it challenging? (Complexity, time pressure, scale)
3. What did you do? (Your specific contributions)
4. What was the result? (Metrics, impact)
5. What did you learn? (Growth)