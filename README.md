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
| 1 | [01_Operating_Systems](./01_Operating_Systems/) | Processes, memory, FS, sync, I/O, Linux kernel |
| 2 | [02_Computer_Architecture](./02_Computer_Architecture/) | CPU, logic, MCU, RTOS, toolchain |
| 3 | [03_Data_Structures_and_Algorithms](./03_Data_Structures_and_Algorithms/) | DS&A, complexity, strings, graphs |
| 4 | [04_Networks](./04_Networks/) | Layers, TCP/IP, HTTP, DNS |
| 5 | [05_Mathematics](./05_Mathematics/) | Discrete math, crypto math, LA, calc, probability |
| 6 | [06_Databases](./06_Databases/) | SQL, transactions, indexes, replication, NoSQL |
| 7 | [07_System_Design_and_Distributed_Systems](./07_System_Design_and_Distributed_Systems/) | Scale, cache, consistency, messaging, APIs |
| 8 | [08_Software_Engineering](./08_Software_Engineering/) | Testing, CI/CD, quality, refactoring |
| 9 | [09_Programming_Languages_and_Runtime](./09_Programming_Languages_and_Runtime/) | Execution, memory, types, concurrency in PLs |
| 10 | [10_Application_Security](./10_Application_Security/) | OWASP, auth, secure coding |
| 11 | [11_Cloud_and_Operations](./11_Cloud_and_Operations/) | Cloud models, Docker/Kubernetes, observability |
| 12 | [12_Theory_of_Computer_Science](./12_Theory_of_Computer_Science/) | Automata, computability, P/NP |
| 13 | [13_Developer_Tools_and_Workflow](./13_Developer_Tools_and_Workflow/) | Git, shell/regex, debug/profile |

### Chapters per track (quick map)

- **01** → `1.1` … `1.8` (8) · **02** → `2.1` … `2.5` (5) · **03** → `3.1` … `3.5` (5)  
- **04** → `4.1` … `4.6` (6) · **05** → `5.1` … `5.5` (5) · **06** → `6.1` … `6.4` (4)  
- **07** → `7.1` … `7.5` (5) · **08** → `8.1` … `8.3` (3) · **09** → `9.1` … `9.4` (4)  
- **10** → `10.1` … `10.3` (3) · **11** → `11.1` … `11.3` (3) · **12** → `12.1` … `12.2` (2) · **13** → `13.1` … `13.3` (3)

---

## How topics connect (navigation)

- **OS → containers**: cgroups/namespaces (**01** / 1.8) → Docker & Kubernetes (**11** / 11.2).
- **Network crypto vs app security**: TLS/PKI (**04** / 4.6) ↔ OWASP & APIs (**10**).
- **DSA ↔ theory**: Big-O (**03**) ↔ P/NP (**12** / 12.2).
- **DB ↔ system design**: SQL & isolation (**06**) ↔ caching, consistency, messaging (**07**).
- **Ship code**: tests & CI (**08**) + Git & debug (**13**) + run in cloud (**11**).

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

---

*Last updated: April 2026*
