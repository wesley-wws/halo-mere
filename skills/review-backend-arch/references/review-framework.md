# Review Framework — Per-Dimension Guidance

Structural guidance for each scoring dimension. Focused on project/module/boundary level.
Does not prescribe code-level patterns.

## 1. Solution & Project Partitioning

**Core question:** Does each .csproj represent a meaningful, independently replaceable unit?

**What to look for:**
- Does the project graph have a recognizable shape (layered, hexagonal, modular) or a tangle?
- Projects that should be one? (e.g., 3 projects with < 5 files that always change together)
- Projects that should be split? (e.g., one project containing both domain logic AND
  infrastructure concerns)
- Is there a "Common" / "Shared" project? Narrow scope → fine. Dumping ground → problem.
- Do test projects mirror source projects?

**Project count heuristic:** Three to four source projects is often enough for a
medium-sized .NET backend (Domain + Application + Infrastructure + Api). Start with
folders inside a project; promote to a separate .csproj only when you need compile-time
dependency enforcement or independent deployment. More projects = more build overhead,
more DI wiring, more cross-project navigation — each boundary must earn its keep.

**Anti-patterns:**
- Monolith in disguise — one project, layers only enforced by folder convention
- Micro-project explosion — 15+ tiny projects for a small app; each boundary is overhead
- Orphan projects — referenced by nothing, serving no clear purpose

**Scoring:**
- 95-100: Each project has clear purpose; clean graph shape; no dump-ground projects
- 80-94: Sound with 1-2 projects that could be consolidated or split
- 60-79: Project boundaries don't align with actual architectural units
- Below 60: No meaningful partitioning, or extreme over/under-partitioning

---

## 2. Domain & Subdomain Boundaries

**Core question:** Do folder/namespace boundaries correspond to real domain concepts?

**What to look for:**
- Can you name each major folder's domain responsibility in one phrase?
- Do folders represent domains (good) or only technical roles (Services/, Repositories/)?
- Are cross-domain interactions explicit (shared interfaces) or implicit (reaching into
  another domain's internals)?
- Is any domain so large it should be split into subdomains?
- **Language divergence test:** If the same concept is named differently in different areas
  (e.g., "User" vs "Buyer" vs "Account"), those may be distinct bounded contexts that
  deserve separate modules.
- **Data isolation:** Do subdomains share database tables? Cross-module foreign keys create
  hidden coupling that makes future decomposition expensive. Each subdomain should ideally
  own its own data, with cross-domain access through explicit lookup interfaces.

**Anti-patterns:**
- Domain soup — one Services/ folder with 20 classes spanning unrelated domains
- Anemic boundaries — folders exist but modules freely import across them
- Wrong seam — organized by technical role instead of by domain
- Missing domain — a significant business concept scattered across multiple modules
- Shared database ownership — multiple subdomains reading/writing the same tables directly

**Scoring:**
- 95-100: Clear domain folders with focused responsibility; explicit cross-domain contracts
- 80-94: Domains identified; 1-2 boundaries fuzzy or one domain overloaded
- 60-79: Some domains mixed; boundaries routinely violated
- Below 60: No domain-based organization

---

## 3. Dependency Direction

**Core question:** Does the project reference graph enforce that high-level policy modules
don't depend on low-level detail modules?

This is about the PROJECT GRAPH, not individual class dependencies.

**What to look for:**
- DAG from .csproj references: arrows point inward (Infrastructure → Application → Domain)?
- Domain project references nothing (or only base libraries)?
- Application references Infrastructure? (It shouldn't — host project bridges via DI)
- Circular references?
- Infrastructure NuGet packages in the wrong project? (e.g., EF Core in Domain)

**Anti-patterns:**
- Shortcut reference — Application → Infrastructure "just for one thing"
- Domain pollution — Domain pulls in infrastructure NuGet packages
- Circular dependency (even transitive)
- Flat graph — all projects reference all others

**Scoring:**
- 95-100: Clean inward-pointing DAG; Domain has zero outward references
- 80-94: Correct direction with minor NuGet leakage
- 60-79: One structural shortcut
- Below 60: Multiple violations or circular references

---

## 4. Module Responsibility

**Core question:** Does each project and major folder have exactly one reason to exist?

**At the project level:**
- Describe each project in one sentence without "and". If you can't → too many responsibilities.
- If a business requirement in one domain changes, how many projects change? If > 2 → leakage.

**At the folder level:**
- Are the 3-5 largest folders justified in size, or have they become catch-alls?
- Folders named "Helpers", "Utilities", "Misc" inside a domain → responsibilities not
  properly placed.

**Anti-patterns:**
- God module — one project/folder owns an entire subsystem with no internal structure
- Responsibility magnet — a module everyone depends on because it does too many things
- Wrong home — a capability in a module where it doesn't belong

**Scoring:**
- 95-100: Every module has clear singular purpose; no catch-all folders
- 80-94: Most focused; 1-2 modules grown beyond original scope
- 60-79: Several modules with blurred responsibilities
- Below 60: No clear responsibility assignment

---

## 5. Entropy Containment

**Core question:** Is complexity concentrated where it naturally belongs?

**What to look for:**
- Complexity gradient: Domain (simplest) → Application (orchestration) → Infrastructure
  (complex I/O)? Or is there a spike in the wrong layer?
- File count distribution: 80% of files in 2 modules might be fine (inherent complexity)
  or a sign boundaries need splitting.
- Cross-cutting concerns: handled architecturally (middleware, filters, DI) or reinvented
  per module?
- State ownership: clear who owns each piece of mutable state? Or multiple modules sharing
  and mutating the same state?

**Anti-patterns:**
- Complexity in the wrong place — business logic in Infrastructure; I/O in Domain
- Distributed state — multiple modules holding pieces of the same conceptual state
- No internal structure — a module with 30+ files and no sub-organization

**Scoring:**
- 95-100: Correct complexity gradient; module sizes match inherent complexity
- 80-94: One module beyond natural bounds but organized internally
- 60-79: Complexity spikes in wrong layer; some modules need internal restructuring
- Below 60: No visible complexity management

---

## 6. Extensibility & Growth Path

**Core question:** Can new features/domains/integrations be added without restructuring?

**What to look for:**
- Adding a new subdomain: is there a clear place for it, or must existing modules change?
- Swapping an infrastructure dependency: how many modules affected?
- Open/Closed at module level: add new behavior by adding modules, not modifying existing?
- Is the architecture prepared for realistic growth (based on project trajectory)?

**Anti-patterns:**
- Shotgun modification — new feature requires touching 5+ modules
- Rigid boundaries — legitimate extensions require restructuring
- No seam for growth — structure works for N domains but has no pattern for N+1
- Over-prepared — elaborate extension points for growth that won't happen

**Scoring:**
- 95-100: Clear pattern for adding domains/features; infrastructure swappable
- 80-94: Growth path exists; one area would require restructuring
- 60-79: New domain would require significant changes to existing modules
- Below 60: Architecture assumes current feature set is final

---

## 7. Architecture Drift & Consistency

**Core question:** Does the structure follow one coherent organizing principle?

**What to look for:**
- All subdomains follow the same folder structure?
- Naming conventions consistent across modules?
- Stated architecture (in docs) matches actual structure?
- "Generation boundaries" — areas where architectural style clearly shifts?
- Cross-cutting concerns have ONE structural solution, or each module invented its own?

**Automated enforcement:**
Architecture rules documented but enforced only by human review will erode over time.
Check whether the project uses architecture fitness functions (NetArchTest, ArchUnitNET,
or custom Roslyn analyzers) to mechanically enforce dependency direction, namespace
organization, and naming conventions. If not, recommend which 2-3 rules to automate first
— typically dependency direction and layer boundary violations, as these have the highest
cost when violated and are trivially testable.

**Anti-patterns:**
- Declared vs. actual — docs say Clean Architecture, code is organized differently
- Convention fragmentation — each subdomain follows its own convention
- Zombie conventions — old patterns nobody follows but not yet removed
- No convention at all — new code has no structural precedent to follow
- Documentation-only governance — rules exist in docs but nothing enforces them; drift is
  inevitable without mechanical checks

**Scoring:**
- 95-100: One consistent principle; all modules follow it; docs match reality; rules
  enforced mechanically where possible
- 80-94: Consistent with 1-2 divergent areas (visible migration direction)
- 60-79: Multiple organizing principles coexist; unclear which is canonical
- Below 60: No consistency — each module is a world unto itself
