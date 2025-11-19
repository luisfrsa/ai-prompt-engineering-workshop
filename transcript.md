Alex (PM): Hey everyone, thanks for making time. Today’s cross‑functional planning session is about permissions on the Employee page and migrating from legacy role_type checks (admin/manager/employee) to explicit user_permission capabilities. We want to leave with a clear v1 scope, concrete view/edit rules per section, and a safe rollout plan.

Alex (PM): Quick recap: admins configure the account and policies, managers run teams day‑to‑day, and individual employees come in to see their profile, assignments, and sometimes update their own info. The Employee page is each person’s profile: basic identity, personal contact, employment details, wages/comp, notes, assignments/tools access, and activity. Permissions here are a top source of bugs and tickets.

Priya (EM): Yep. For engineering success means fewer access bugs, less mobile/web drift, and a model we can extend without spelunking through role_type if‑elses.

Ann (Dev): And a single source of truth. Right now role_type check is overloaded and duplicated across backend, frontend, and mobile.

Marco (Dev): I’m new in this code, so I’ll ask clarifiers. I saw a lot of “if admin or manager” checks that don’t match each other.

Nina (Designer): I’ll focus on how it feels in the UI: what’s visible, what’s read‑only, and what gets hidden or blocked.

Alex (PM): Great. Before we get into rules, here’s a real world moment from CS: A typical  manager at Customer “BrightLeaf” logs in Monday morning, opens an employee profile to update their title and team after a reorg, then checks wages for direct reports before payroll. They assume “if I can see the person, I can edit their work info,” and when that fails we get tickets. We need to match those expectations or explain clearly.

Priya (EM): That’s helpful framing.

Alex (PM): Cool. Let’s list Employee page sections as users see them: (1) top summary card with name/photo/title/team/location/status, (2) Personal Info tab: address, phone, emergency contacts, plus role and basic identifiers, (3) Employment Details tab: start date, job type, manager, location assignments, termination fields, (4) Wages & Compensation tab: current rate/salary, history, effective dates, (5) Notes tab: manager notes and HR private notes, (6) Assignments tab: projects and internal tool access, (7) Activity log tab. Actions include edit profile, change role, deactivate/reactivate, transfer teams, and send invite/reset password.

Ann (Dev): Activity log is partly pulled from audit‑service, but UI is a tab on Employee page.

Alex (PM): Perfect. Now section by section for view vs edit.

Alex (PM): Personal Info — viewing. We have org visibility settings. If “Org directory” is enabled, any employee can see basic profile fields for others: name, title, team, location. Private personal fields (home address, phone, emergency contact) should be restricted. Proposal: 
- can_view_basic_profile: all employees if directory on; otherwise admins/managers only.
- can_view_private_personal: admins; managers for people in their management tree; employees for self.

Nina (Designer): That matches the current directory behavior.

Marco (Dev): When you say management tree, that includes indirect reports?

Priya (EM): Yes, same as today. Assistant managers are an edge case, but not v1.

Alex (PM): Right — v1 scope keeps current “tree” logic. Assistant managers stay as current behavior. We’ll flag as open question.

Alex (PM): Personal Info — editing. 
- Employees can edit their own contact/emergency contact (can_edit_self_personal).
- Admins can edit any personal fields (can_edit_private_personal).
- Managers can edit “work‑related basics” for their tree (title, team, location) but not home address (can_edit_basic_profile scoped).

Nina (Designer): I’ll group “Work info edits” together so managers don’t see edit affordances on private rows.

Ann (Dev): We’ll need field‑level backend enforcement too, not just UI.

Alex (PM): Noted, that’s a recurring risk.

Alex (PM): Employment Details — viewing. 
- can_view_employment_details: admins, managers over tree, and employees for self.
- Termination reason stays admin‑only via can_view_termination_reason.

Ann (Dev): Termination reason sits in the same object; we must filter server‑side.

Priya (EM): Yes, because hiding in UI alone risks leakage.

Alex (PM): Employment Details — editing.
- can_edit_employment_details: admins for anyone.
- Managers can edit team assignments and manager assignment within tree (can_edit_team_assignments).
- Employees can’t edit these in v1.

Marco (Dev): If a manager edits another manager’s team, does validation prevent cycles?

Ann (Dev): Usually, but it’s legacy. v1 keeps behavior; we’ll add tests where possible.

Alex (PM): Wages & Compensation — viewing. Here’s where expectations vary. Base behavior:
- Admins with wage access can view.
- Some managers with wage access can view wages for their tree.
- Employees generally can’t see wages, except rare accounts with “self wages visibility.”

Marco (Dev): I assumed employees could see their own wages by default. Thanks for clarifying.

Nina (Designer): When self wages is on, the wages tab appears just for self with privacy copy.

Alex (PM): So v1 capabilities:
- can_view_wages (tree‑scoped unless admin),
- can_edit_wages (admin/explicitly granted),
- can_view_own_wages (only if setting enabled).

Ann (Dev): History list and edit modal currently check different legacy flags. This migration should unify them.

Priya (EM): And backend must enforce so mobile doesn’t leak even if it lags.

Alex (PM): Notes — viewing. There are two note types:
- Manager Notes (for performance/1:1 context),
- HR Private Notes (sensitive).

Nina (Designer): UI shows one Notes tab with two sections.

Marco (Dev): So managers can see both sections for their reports?

Ann (Dev): Small correction — managers can see only Manager Notes today. HR Private Notes are admin/HR only.

Marco (Dev): Got it, thanks. That’s our second misunderstanding.

Alex (PM): Great catch. v1:
- can_view_manager_notes: managers over tree + admins.
- can_view_hr_notes: admins only.
- can_edit_manager_notes: managers/admins.
- can_edit_hr_notes: admins only.
And importantly, backend filters HR notes out entirely unless you have can_view_hr_notes.

Priya (EM): Again, data leakage risk. We should call it out clearly across roles.

Alex (PM): Yup — I’ll repeat it in the recap.

Alex (PM): Assignments — viewing/editing.
- can_view_assignments: admins + managers over tree; employees for self for their own assignments.
- can_edit_assignments: admins only for v1. We’ll keep existing special cases mapped later.

Ann (Dev): Works. Assignments are already mostly admin‑only.

Alex (PM): Activity log — viewing.
- can_view_activity_log: admins any; managers for tree; employees for self.
No edits.

Nina (Designer): If you lack access, I’d rather show the tab with a blocked message than make it vanish.

Marco (Dev): I was thinking hide to reduce clutter, but blocked makes sense for transparency.

Alex (PM): Agreed. Rule of thumb for UX:
- Core tabs (Personal, Employment, Activity): always visible; fields/tables are read‑only or blocked based on can_view.
- Sensitive optional tabs (Wages, Notes, Assignments): visible if feature enabled; blocked if no access; hidden only if feature itself off.
- Add‑on “Documents” tab stays legacy and hidden unless enabled + access.

Nina (Designer): Great. I’ll update prototypes for those patterns.

Alex (PM): Now migration from role_type to capabilities. Ann, where are legacy checks hiding?

Ann (Dev): Three places: backend policies in monolith, frontend roles utilities, and some inline checks in GraphQL resolvers; plus mobile duplicates pieces. We need a bridging layer that computes capabilities from role_type + account settings + existing feature flags. Then we replace checks gradually.

Priya (EM): Plan for safety: dual‑read. Keep role_type for compatibility, compute capabilities for everyone, and introduce a feature flag that switches enforcement to permissions once we’re confident.

Alex (PM): Any big legacy surprises?

Ann (Dev): Lots of small special cases. Example: some customers have “manager_can_edit_profile” override or pseudo HR roles. If we don’t map those, we’ll regress behavior.

Marco (Dev): So we need a mapping doc and an audit of existing checks.

Priya (EM): And tests. Current coverage is thin; shadow mode with metrics will help, but we still need unit tests per sensitive field.

Alex (PM): I want to highlight the cross‑cutting risk explicitly: data leakage. Ann flagged it for termination reason and HR notes; Priya flagged backend filtering; and from product POV, leakage is a trust‑killer. So v1 must enforce sensitive filtering in backend even if UI or mobile is behind.

Ann (Dev): Agreed. Backend enforcement is non‑negotiable.

Alex (PM): v1 scope proposal:
1) Define/seed Employee page capabilities we listed.
2) Backend computes capabilities from legacy and enforces filtering for sensitive data (wages, HR notes, termination reason).
3) Web Employee page swaps role_type checks for can_* capabilities behind a feature flag.
4) Mobile does not migrate UI in v1 but consumes a minimal capability list to hide/block tabs; backend filtering protects data.
5) Documents add‑on and custom roles editor are out of scope; we preserve existing behavior by mapping their current flags.

Priya (EM): That’s realistic for one iteration.

Nina (Designer): Another user story moment from research: At Customer “Lighthouse,” an engineer‑turned‑manager uses Employee profiles mostly to see who’s on call and what tools they have. They rarely touch wages, but they do rely on Manager Notes for performance context. If those notes disappear or show extra HR content, that’s a big trust issue. So the blocked vs hidden behavior on Notes really matters.

Alex (PM): Perfect, thanks.

Marco (Dev): For mobile flags, are we sending per‑tab booleans or a list of capabilities?

Ann (Dev): We already send a small “permissions” bag; I’ll spike what mobile consumes and propose a minimal contract like can_view_wages, can_view_notes, can_view_assignments, can_edit_basic_profile.

Priya (EM): Great. Ann owns that spike.

Alex (PM): Rollout: Priya’s shadow mode idea sounds right. We compute capabilities for all, run new checks in shadow, log diffs between legacy and new decisions, dogfood internally, then beta with a handful of friendly customers already on the new Employee layout.

Alex (PM): I’ll work with CS/Sales to pick 5 beta accounts and set expectations.

Priya (EM): We’ll also need a rollback path: keep feature flag so we can revert to legacy instantly if diffs spike.

Alex (PM): Open questions / risks:
- Mapping gaps from legacy special cases to capabilities (regression risk).
- Data leakage if backend filtering misses a field.
- Limited tests; shadow mode might not catch rare paths.
- Edge cases like assistant managers.
- Billing dependency: can_manage_billing.

Priya (EM): On billing: some Employee actions — promoting to admin or deactivating last admin — have billing invariants. Today they piggyback on role_type=admin. We need to confirm with Billing/Payments team how can_manage_billing should be assigned and enforced. No billing rep here, so this ends as follow‑up.

Ann (Dev): If we expose the “Make admin” action based on wrong permission, users could open a modal and get blocked by billing service. Bad UX.

Nina (Designer): I’ll gate the affordance on can_manage_billing once semantics are confirmed.

Priya (EM): I’ll add a small engineering task to ensure GraphQL resolvers stop doing inline role_type checks for these fields; they should reference the new policy layer.

Ann (Dev): I can fold that into backend work.

Marco (Dev): Anything else to clarify on scope?

Alex (PM): Just to restate: v1 is “same behavior, clearer model, safer enforcement.” We’re not redesigning roles UI or changing how assistant managers work yet.

Nina (Designer): Works for me.

Priya (EM): Same.

Alex (PM): Awesome. I’ll post recap after this. Thanks everyone — good progress.