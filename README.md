# Computer Science Study Repository

Study notes and topic maps for computer science and software engineering interviews. Each numbered folder is a **track**; inside it, **`N.M_*` folders** are chapters, each with a **`README.md`** outline.

---

## Layout convention

| Level | Pattern | Purpose |
|-------|---------|---------|
| Track | `NN_Snake_Case_Name/` | Major area (01–13) |
| Chapter | `N.M_Short_Name/` | Subtopic; `N.M` matches the parent track number |
| Entry point | `README.md` | Scope, bullets, study/practice checklists |

**Optional folders** (add when you start that topic): `notes/`, `resources/`, `exercises/`, `projects/` — same idea in any chapter folder.

---

## Track index

The full map of tracks lives **only here**; individual `README.md` files under each track do not repeat a “back to index” footer.

| # | Folder | Focus |
|---|--------|--------|
| 00 | [00_Developer_Tools_and_Workflow](./00_Developer_Tools_and_Workflow/) | Git, shell/regex, debug/profile |
| 01 | [01_Mathematics](./01_Mathematics/) | Discrete math, crypto math, LA, calc, probability |
| 02 | [02_Data_Structures_and_Algorithms](./02_Data_Structures_and_Algorithms/) | DS&A, complexity, strings, graphs |
| 03 | [03_Computer_Architecture](./03_Computer_Architecture/) | CPU, logic, MCU, RTOS, toolchain |
| 04 | [04_Theory_of_Computer_Science](./04_Theory_of_Computer_Science/) | Automata, computability, P/NP |
| 05 | [05_Operating_Systems](./05_Operating_Systems/) | Processes, memory, FS, sync, I/O, Linux kernel |
| 06 | [06_Programming_Languages_and_Compilers](./06_Programming_Languages_and_Compilers/) | Execution, memory, types, concurrency, COMPILERS |
| 07 | [07_Networks](./07_Networks/) | Layers, TCP/IP, HTTP, DNS |
| 08 | [08_Databases](./08_Databases/) | SQL, transactions, indexes, replication, NoSQL |
| 09 | [09_Application_Security](./09_Application_Security/) | OWASP, auth, secure coding |
| 10 | [10_Software_Engineering_and_Architecture](./10_Software_Engineering_and_Architecture/) | Testing, CI/CD, quality, refactoring, DESIGN PATTERNS |
| 11 | [11_System_Design_and_Distributed_Systems](./11_System_Design_and_Distributed_Systems/) | Scale, cache, consistency, messaging, APIs |
| 12 | [12_Cloud_and_Operations](./12_Cloud_and_Operations/) | Cloud models, Docker/Kubernetes, observability |
| 13 | [13_Artificial_Intelligence_and_Machine_Learning](./13_Artificial_Intelligence_and_Machine_Learning/) | Machine Learning, Deep Learning, MLOps |

### Chapters per track (quick map)

- **00** → `0.1` … `0.3` (3) · **01** → `1.1` … `1.5` (5) · **02** → `2.1` … `2.5` (5)
- **03** → `3.1` … `3.5` (5) · **04** → `4.1` … `4.2` (2) · **05** → `5.1` … `5.8` (8)
- **06** → `6.1` … `6.4` (4) · **07** → `7.1` … `7.6` (6) · **08** → `8.1` … `8.4` (4)
- **09** → `9.1` … `9.3` (3) · **10** → `10.1` … `10.4` (4) · **11** → `11.1` … `11.5` (5)
- **12** → `12.1` … `12.3` (3) · **13** → `13.1` … `13.3` (3)

---

## How topics connect (navigation)

- **OS → containers**: cgroups/namespaces (**05** / 5.8) → Docker & Kubernetes (**12** / 12.2).
- **Network crypto vs app security**: TLS/PKI (**07** / 7.6) ↔ OWASP & APIs (**09**).
- **DSA ↔ theory**: Big-O (**02**) ↔ P/NP (**04** / 4.2).
- **DB ↔ system design**: SQL & isolation (**08**) ↔ caching, consistency, messaging (**11**).
- **Ship code**: tests & CI (**10**) + Git & debug (**00**) + run in cloud (**12**).

These are reading hints, not a mandatory order.

---

## How to use this repository

1. Open a track **`README.md`** for the chapter list, then each chapter’s **`N.M_*/README.md`**.
2. When you deepen a topic, add optional folders under that chapter as described in [Layout convention](#layout-convention).
3. Follow cross-links between tracks when one topic spans several areas.

---

## Study guidelines

- Write technical notes in **English** when possible (easier to match docs and interviews).
- Prefer small, updated commits over one-off dumps.
- Review and refresh **`README.md`** checklists as you finish sections.
- Treat every checklist item as incomplete until you can do at least two of: derive/prove it, implement it, measure it, debug a failure case, or compare it against a real system.

## Depth standard

A topic is not "done" when you have read a definition. Mark it as solid only when your notes include:

- **Concept**: precise definitions, assumptions, and common misconceptions.
- **Mechanism**: how it works internally, preferably with diagrams or state transitions.
- **Proof or analysis**: correctness argument, complexity bound, invariant, or formal trade-off.
- **Implementation**: a small program, simulation, lab, or reproduction.
- **Measurement**: benchmark, trace, profiler output, packet capture, query plan, or hardware counter when relevant.
- **Failure mode**: a bug, outage pattern, security issue, race, anomaly, or edge case.
- **Primary source**: textbook chapter, RFC, paper, vendor manual, language spec, or official documentation.

---

*Last updated: April 2026*
