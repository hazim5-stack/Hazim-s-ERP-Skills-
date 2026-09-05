# The 2026 AI-Built ERP Debate — Why This Repository Exists

## Author's note

I am **Hazim Batwa**. This repository was created after I studied a public engineering debate that followed an ERP demonstration posted by Mahmoud Sulaiman (@mahmooud_2009) on 27 August 2026.

My purpose here is not to attack the person who built the prototype. In fact, the episode is interesting precisely because it showed how far a non-career software engineer can now go with AI. My interest is the engineering boundary between **a convincing ERP interface** and **an operational system that can safely become the source of truth for a company**.

---

## What triggered the debate

The post presented an ERP-style system for a factory and compared the rapid AI-assisted build with an external implementation quote of 80,000 riyals. The screenshot showed familiar manufacturing and enterprise modules such as purchasing, inventory, quality, finance, HR, production and sales.

The engineering community immediately focused on what could not be proved by a screenshot.

A dashboard can prove that a dashboard exists. It cannot prove that:

- two concurrent reservations cannot oversell the same stock,
- received goods are recognized correctly before the supplier invoice,
- FIFO is really implemented as cost layers,
- inventory and accounting post atomically,
- closed accounting periods cannot be silently rewritten,
- rejected quality stock cannot be shipped,
- tenant boundaries survive direct API calls,
- retrying after a timeout cannot duplicate a financial transaction,
- reports reconcile to ledgers after months of operation,
- backups can actually restore the business to a known point in time.

---

## The two examples that changed the discussion

The strongest lesson came from follow-up technical discussion around two domain behaviors.

### 1. Goods receipt versus supplier invoice

Physical receipt and supplier invoicing are separate business events. A serious purchasing/accounting design must explicitly model the received-not-invoiced state — commonly handled through GRNI (Goods Received Not Invoiced) or an equivalent accounting policy.

If the system recognizes inventory only when the supplier invoice arrives, physical stock and financial stock can diverge. That is not a visual defect. It is a wrong event model.

### 2. FIFO shown, moving average executed

If a user can select FIFO, the system must actually maintain and consume FIFO cost layers according to defined rules.

Silently using moving average while showing FIFO is particularly dangerous because the workflow appears successful. The system does not crash. It simply produces financially incorrect meaning.

That became one of the central principles behind these skills:

> **Silent success can be more dangerous than an explicit failure.**

---

## What I took from the debate

The useful conclusion is not that AI cannot build enterprise software.

AI can already accelerate:

- UI implementation,
- API scaffolding,
- schema drafts,
- migrations,
- validations,
- automated tests,
- CI configuration,
- documentation,
- refactoring,
- code review.

The problem is that the model is often strongest at reproducing recognizable software structure and weakest when the required rule is implicit inside accounting, warehouse operations, production, quality, law, internal controls or exception handling.

ERP software is dominated by those rules.

A factory or company does not need a system that only knows how the happy path looks. It needs one that remains correct when:

- documents are partial,
- dates are backdated,
- people make mistakes,
- the network times out,
- a worker retries,
- two users act simultaneously,
- a supplier invoices late,
- an item is quarantined,
- a production order is partially completed,
- an integration fails after local commit,
- the database deadlocks,
- a period has already closed,
- historical reports are regenerated months later.

---

## Why I created two skills instead of one

I separated the engineering problem into two different AI roles.

### Planning Skill

The planning skill exists to stop the AI from jumping from a requirement directly to screens and tables.

It asks first:

- what business event occurred?
- what fact becomes true?
- which ledger owns that fact?
- what invariant must never be violated?
- what permissions apply?
- what happens under concurrency?
- what happens after a timeout?
- how is the event reversed?
- how does it reconcile to accounting and reporting?
- what test proves the rule?

### Engineering Audit Skill

The audit skill assumes the implementation already exists and becomes adversarial.

It does not accept architecture vocabulary as proof. It looks for repository, schema, migration, runtime, test, deployment and recovery evidence.

If evidence is absent, the correct result is not PASS. The correct result is **NOT TESTED**.

---

## My position on AI-assisted development

I created this project because I want to use AI to build more software, not less.

But I do not want speed to erase engineering discipline.

The new bottleneck is increasingly not typing code. It is defining the contracts the generated code is not allowed to violate.

For ERP, those contracts include:

```text
balanced journals
atomic stock/accounting transitions
real costing semantics
closed-period rules
idempotent business commands
segregation of duties
tenant isolation
traceable reversals
reconciliation
recoverability
```

The AI should be allowed to move fast **inside those boundaries**.

That is the purpose of Hazim's ERP Engineering Skills.

— **Hazim Batwa**
