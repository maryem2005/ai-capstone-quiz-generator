## Additional Limitations, Design Tradeoffs, and Engineering Reflections

### Research Limitations: Existing Airtable Quiz Implementations Were Not Suitable

Before implementing the quiz delivery experience, I researched existing Airtable quiz workflows through documentation and YouTube tutorials to understand best practices for building a quiz interface within Airtable.

A major limitation quickly became apparent: most publicly available Airtable quiz implementations were designed for static quizzes.

These implementations typically assumed:

- a fixed set of prewritten questions
- the same quiz content for every user
- manually curated question banks
- no AI-generated content
- minimal workflow automation

In many examples, users were still expected to manually select which questions they wanted to answer before beginning the quiz.

This design was not appropriate for our project.

Our system required:

- dynamically generated quizzes
- changing question sets on each execution
- AI-generated content that differs every run
- automated delivery tied directly to uploaded documents

Because of this, much of the existing Airtable guidance was not directly transferable.

This meant a significant amount of the quiz delivery architecture had to be designed experimentally rather than following an existing implementation pattern.

---

### Airtable Was Not Designed as a True Quiz Platform

Although Airtable worked effectively as a backend database and lightweight interface, it was not ideal as an interactive quiz platform.

Key limitations included:

- long question text being visually truncated
- limited control over layout and presentation
- difficulty hiding backend implementation fields cleanly
- awkward form-based interaction compared to traditional quiz interfaces
- limited ability to create dynamic, question-by-question interaction

Considerable effort went into improving usability by:

- hiding technical fields such as `record_id`, `status`, and linked record references
- reorganizing visible layouts
- simplifying the user-facing experience as much as possible

Even after refinement, Airtable remained a compromise rather than an ideal frontend solution.

---

### Major Response Model Redesign for User Experience

One of the most important architectural decisions involved redesigning how user quiz submissions were handled.

The original implementation was structured around one response record per question.

This meant the intended logic looked like:

- user answers Question 1
- one response record is created
- workflow grades that response
- user answers Question 2
- another response record is created
- repeat for every question

Although technically normalized from a database perspective, this created a poor user experience and significantly increased workflow complexity.

Problems included:

- too many records being created per quiz attempt
- more complicated grading logic
- harder record linking
- more workflow overhead
- less intuitive quiz submission for the user

To improve usability, the submission model was redesigned.

Instead of processing one question at a time, the final implementation processes one complete quiz attempt containing all submitted answers.

The updated workflow logic became:

1. Trigger when a single Responses form submission is received
2. Read the selected quiz
3. Pull the linked questions associated with that quiz
4. Map submitted answers to the corresponding questions:
   - `answer_1 → Question 1`
   - `answer_2 → Question 2`
   - ...
   - `answer_10 → Question 10`
5. Compare each submitted answer against the correct answer
6. Calculate total grading results
7. Generate feedback and performance analytics

This changed the architecture from:

**one response per question**

to:

**one response per quiz attempt**

This was a major simplification that improved both usability and workflow maintainability.

---

### Unused "Source" Field Reflection

One of the original schema design decisions was to require a `source` field in every Airtable table for consistency and traceability.

The intention was that this field would indicate how a record entered the system, such as:

- upload
- API
- AI-generated
- manual entry
- system-generated

However, during implementation, this field was never meaningfully used.

Despite being part of the original design standard, the workflows functioned successfully without relying on it.

This became an important design reflection.

Lesson learned:

Not every theoretically useful field provides practical implementation value.

In future iterations, schema design would benefit from prioritizing fields based on actual workflow dependencies rather than architectural assumptions alone.

---

### Dynamic AI Generation Made Traditional Testing Harder

Because questions are generated dynamically by AI, outputs change between runs.

This created several testing challenges:

- questions could not be assumed to remain identical
- expected outputs changed every execution
- debugging required checking structure rather than exact content
- repeated testing introduced natural content variation

This made validation fundamentally different from testing deterministic software systems.

The focus shifted from checking exact outputs to validating:

- schema correctness
- answer formatting
- explanation generation
- difficulty assignment
- field population
- linkage consistency

---

### Flowise Prompt Variable Sensitivity

The integration between n8n and Flowise required exact prompt variable alignment.

Small mismatches caused failures.

Examples included:

- undefined prompt variables
- incorrect JSON references
- missing `notes`
- wrong node outputs being referenced
- incorrect field mappings

Because Flowise requires expected prompt variables to exist exactly as configured, even minor naming inconsistencies caused full execution failures.

This significantly increased debugging complexity.

---

### Flowise Endpoint Dependency

Another unexpected integration challenge involved Flowise deployment behavior.

Whenever a Flowise chatflow was modified or redeployed, the endpoint URL used inside n8n could change.

If the HTTP Request node continued referencing an outdated endpoint, the integration would fail despite the underlying logic being correct.

This created a hidden dependency between workflow maintenance and deployment state.

---

### Metadata Propagation Complexity

A major recurring engineering challenge involved metadata propagation.

Important fields frequently needed to move across multiple workflows, including:

- document references
- quiz references
- response references
- generation IDs
- timestamps
- user identifiers
- topic labels
- performance links
- confidence scores

If any stage failed to carry these values forward correctly, traceability suffered.

This reinforced the importance of designing metadata flow intentionally rather than assuming downstream inheritance.

---

### Linked Record Dependency Complexity

Airtable’s linked record structure was useful for maintaining relational integrity, but it also introduced implementation complexity.

The system required correct links between:

- Documents → Questions
- Questions → Quizzes
- Quizzes → Responses
- Responses → Performance

If any link failed:

- grading broke
- dashboard reporting became incomplete
- record tracing became harder
- analytics accuracy decreased

Managing relational consistency across low-code tools required more coordination than initially expected.

---

### Automation Was Limited by Platform Constraints

The final workflows functioned correctly when manually executed.

However, publishing and fully automating execution was blocked by n8n’s execution limits/paywall constraints.

As a result:

- workflows required manual triggering
- full autonomous execution could not be demonstrated

This was an infrastructure limitation rather than a workflow design failure.

[Insert screenshot of n8n execution limitation here]

---

### Low-Code Tools Accelerated Development but Added Constraints

Using Airtable, n8n, and Flowise significantly accelerated prototyping.

However, low-code tools also introduced hidden constraints:

- interface limitations
- execution quotas
- brittle integrations
- restricted customization
- dependency on vendor platform behavior

These tradeoffs became increasingly visible as the project matured.

---

### Team Dependency Risk

The original architecture assumed each component would be completed sequentially by different team members.

This created dependency risk.

If one component stalled:

- downstream development paused
- checkpoints became blocked
- integration testing was delayed
- architectural issues remained hidden longer

This became a major project management lesson.

Future implementations would benefit from earlier integration testing and more modular ownership boundaries.
