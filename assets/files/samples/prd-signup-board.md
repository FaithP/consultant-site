# Product Requirements Document — Presenter Signup Board

*Working title. The name is a deliberate open item; see Open Questions.*

A lightweight, two-sided web app that lets an organizer manage a rotating set of voluntary presenters across recurring sessions — and lets people claim a slot in seconds, with no account.

---

## 1. Problem

A recurring critique-circle format runs short sessions with a small, fixed number of presentation slots. The rougher the work, the better — the format exists to lower the bar for putting unfinished things in front of others.

Today the organizer manages this *somehow*: people post in chat, send DMs, or email to volunteer, and the organizer keeps a list by hand. This is fine at two presenters and miserable at twelve. It has no notion of order, no waitlist, no way for a volunteer to see where they stand, and no graceful way to handle the last-minute backout — the failure mode that actually happens.

Underneath the specific case is a general pattern: **a fixed set of scarce slots, a population of people who want to occupy them, and a host who has to curate the match.** The same structure recurs in auditions, open-mic nights, office-hours sign-ups, potluck dish assignments, and booth reservations. This product solves the narrow case well and is built so the pattern can be extended later — but it does not try to be the general tool on day one.

## 2. Goals (v1)

- Let a volunteer claim a slot with near-zero friction: a link, a name, an email, and done.
- Give the organizer real control over the running order — not just a list, but a *setlist* they can reorder and reshape live.
- Keep participation low-pressure, in keeping with a "rough work welcome" culture.
- Be walkable end-to-end by a stranger in about a minute (this is also a portfolio requirement).

## 3. Non-goals (v1)

- Not a scheduling tool with timed slots, durations, or buffers (that is a different product — see Deferred).
- Not a membership system. Volunteers do not create accounts.
- Not an automated notification system. No digests, no scheduled email.
- Not a multi-session orchestrator. Events are independent; nothing rolls over or leapfrogs between them.

## 4. Users & roles

**Organizer** — the only real account in the system. Creates and configures events, opens and closes signups, shares the join link, and curates the setlist. Higher reliability needs than volunteers (they have to get in five minutes before a session starts), which justifies a more robust sign-in.

**Volunteer (signer-upper)** — link-native, no account. Reaches the right event through a shared join link, signs up, and receives a private link that is simultaneously their confirmation, their edit handle, and their live status page. Possession of that link *is* their identity.

## 5. Product structure — two faces

The app has two surfaces that share almost no navigation, because they serve two audiences who rarely cross over:

- **Public surface** — a linear flow: open the join link → see the event and the board → sign up → land on your personal page. There is no reason for a volunteer to navigate "back" to a blank form.
- **Organizer surface** — an authenticated dashboard reached through a different door (sign-in), where events are configured and setlists are curated.

The build is mobile-first; specific layout and navigation are left to implementation rather than prescribed here.

## 6. Core concept & promotion policy

The board is a single primitive: **a pool of entries and a set of N scheduled slots that some entries are promoted into.** What looks like several different "modes" (first-come, organizer-pick, voting) is really one board with a swappable promotion *policy*. v1 ships exactly one policy:

**First-come, first-served, with an opt-out.** New entries fill open scheduled slots in order. Once slots are full, further entries go to the waitlist. The one wrinkle: a volunteer may *choose* the waitlist even when a slot is open — useful for someone who wants in but not this time.

## 7. Feature set (v1)

**Organizer can:**

- Create and configure an event: name, date, time, location, number of scheduled slots, waitlist on/off, and list visibility.
- Run more than one independent event (no coupling between them).
- Open or close signups. The shareable join link only exists once the event is open — so organizers can build in private and expose when ready.
- Curate the setlist live: reorder scheduled presenters, remove an entry, and **promote a waitlisted person into a slot, which automatically demotes the displaced presenter to the top of the waitlist.** This is the heart of the tool — the difference between a real app and a to-do list — and is where polish should concentrate.
- See each volunteer's email and a copyable link to each volunteer's personal page (both organizer-visible; neither is ever shown on the public board).

**Volunteer can:**

- Reach the correct event via a join link.
- Sign up with a name and email (both required), a short presentation title (optional but encouraged), and an optional description.
- On Save, land on a **personal page** that doubles as confirmation, shows their current status ("Presenter 2," "Waitlist 3"), and lets them edit or remove their own entry — and *only* their own.
- Copy their personal link or email it to themselves; the page is a stable URL they can bookmark or save anywhere.

**Shared mechanics:**

- **Occupancy is always visible** — "4 of 6 filled, 2 on the waitlist" — which preserves a gentle scarcity nudge and helps with the "nobody wants the last slot" hesitation.
- **Identity/content visibility is organizer-controlled**, defaulting to *content visible, names hidden*. This keeps the board looking alive while removing the personal-comparison sting that deters hesitant first-timers in a rough-work-welcome culture.
- **Waitlist is on/off only.** When on, its size is set automatically (roughly 50–75% of scheduled slots); the organizer sees the computed size but does not set it. One fewer control to build and to think about.

## 8. Key product decisions & rationale

These are the decisions that distinguish this from a form with a list attached.

**One board, one policy setting — not three apps.** First-come / organizer-pick / voting are the same data and the same controls under different promotion rules. v1 ships one policy and leaves the setting point in place for later.

**The edit token, confirmation, and status page are one object.** Saving an entry mints a single unguessable secret; that secret is the URL of the personal page. There is no separate human-readable confirmation code. This gives real per-entry ownership — the "mean girl crosses off a rival" problem is structurally impossible, because acting on an entry requires its token — without any account system.

**Email is required, and it pays for itself twice.** It gives the organizer a contact channel for presenters, and it doubles as a manual recovery path for a lost token link: because the organizer can see and copy each entry's personal-page link, a volunteer who loses theirs can simply be given it back. This is why automated confirmation email is *deferred rather than missing* — it automates a recovery the organizer can already perform by hand.

**"Closed" means one thing: no new signups.** Closing does not archive the event or freeze the board. Status keeps moving (organizers reorder and promote after close), personal pages stay live, and presenters can still withdraw — which the organizer needs to see as a gap. Closing locks the front door, not the house.

**Anonymity by default is a values choice, not an oversight.** A visible roster of who-is-presenting-what creates comparison anxiety precisely in the courage-gathering moment the format is built to ease. Showing *what* without *who* keeps the board legible and alive while protecting the hesitant. The organizer can override per event.

**Entry links must be unguessable.** A sequential or predictable identifier would turn the personal-page URL into an open door to edit strangers' entries. The implementation chooses the scheme; the requirement is that the per-entry secret be long and random.

## 9. Domain model (conceptual)

Three entities. An **Organizer** owns zero or more **Events**. An **Event** holds configuration (name, date, time, location, slot count, waitlist on/off, visibility, open/closed state) and a join link, and contains an ordered set of **Entries**. An **Entry** holds a name, email, presentation title, optional description, its position (which scheduled slot, or which waitlist position), and its unguessable secret. Promotion and reordering operate on Entry position within an Event; no relationship spans events.

## 10. Out of scope — deferred, with reasons

Most of these solve the *overflow* problem (more demand than the format can yet attract) or add coupling between events. They are anticipated, not foreclosed.

- **Membership / approval model** (volunteers create accounts, await organizer approval). Deferred until automated spam is a real problem; an unguessable join link plus basic rate-limiting covers the realistic threat.
- **Automated email** — confirmations, daily digests, night-before final list. Deferred; manual recovery via captured email suffices at community scale.
- **Voting as a promotion policy.** The board already supports it conceptually; not built.
- **Integrations** — Slack / Zoom / Meet for in-meeting agendas or out-of-app management. The highest-cost, highest-risk surface; deferred.
- **Inter-event logic** — auto-rollover, leapfrog signups for a future session, pull-leftover-presenters-to-next-event, cross-event signup. All deferred; this is the expensive coupling the independent-events model deliberately avoids.
- **Save-settings-as-default** for organizers running many events. A preference-persistence layer serving a future that does not exist yet.
- **Event lifecycle** — archiving, wrap-up, "can't open a new one until you close the current."
- **Mixed virtual / in-person presenters.**
- **Timed-slot variant** (job-fair scheduling, office hours with durations and buffers). A *cousin*, not a child — it is Calendly's shape, not this app's, and is noted to keep the v1 data model from over-generalizing toward it.
- **Broader brand extensions** — auditions, open-mic nights, potluck assignments, gala and fundraising-walk signups, booth reservations, timed-entry events, book-club host/pick rotations, shared meal-plan cooking rotations. The resource-slot pattern generalizes; v1 does not.

## 11. Technical approach

Built on Lovable (React front end, Supabase backend for auth, database, and hosting). Organizer authentication uses Supabase email + password as the primary method, with Google sign-in as a welcome bonus if it comes nearly free and skipped otherwise. Volunteers use no auth — only the join link and their per-entry secret.

## 12. Success criteria

- A stranger can open the join link and complete a signup in under a minute, then find their status again later from their saved link.
- An organizer can absorb a last-minute backout — promote a waitlister, watch the displaced presenter move to the top of the waitlist — in a few clicks.
- The board reads as a real, populated application on first glance, without exposing anyone to comparison anxiety they didn't ask for.

## 13. Open questions

- **Name.** Intentionally unresolved. The working title is descriptive, not final.
