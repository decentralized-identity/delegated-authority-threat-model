# Threat Model for Delegated Authority in AI-Mediated Systems

*A Governance Lens on Failure Modes*

Specification Status: Editor's Draft

Latest Draft:
[identity.foundation/delegated-authority-report](https://identity.foundation/delegated-authority-report)

Ratified Versions:

Editors:

- Debbi Bucci (@debbucci)

Contributors:

- Debbie Bucci (@debbucci)
- Juan Caballero (@bumblefudge)
- Alan Karp (@alanhkarp)

Participate:
~ [GitHub repo](https://github.com/decentralized-identity/governance-of-delegated-authority-report)
~ [File a bug](https://github.com/decentralized-identity/governance-of-delegated-authority-report/issues)
~ [Commit history](https://github.com/decentralized-identity/governance-of-delegated-authority-report/commits/main)
~ [Working Group](https://identity.foundation/working-groups/trusted-agents.html)

Except where otherwise noted, this work by the [Decentralized Identity Foundation](https://identity.foundation/) is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0).

## Related documents

* [Delegated Authority Report](https://identity.foundation/delegated-authority-report/)
* [Governance of Delegated Authority Report](https://identity.foundation/governance-of-delegated-authority-report/)
* [Demo of a Delegation User-Story](https://www.youtube.com/watch?v=u-uWl_s0PPM%20)


## Abstract

Delegated authority in AI-mediated systems fails not at the moment of issuance but at the moment of action. This paper catalogs five structural failure modes — authority not properly *bounded, attested, enforced, authenticated,* or *interpreted* — that emerge when governance frameworks designed for human-paced execution are applied to autonomous agents operating across organizational and technical boundaries. All five are amplified by time: authority valid at issuance may not remain valid, traceable, enforceable, secure, or semantically aligned at execution. These failures represent a class of Governance TOCTOU vulnerability — the authority checked at issuance is not the authority present at execution.

This paper introduces “subject agency state” as a first-class governance variable: whether the person behind a delegation can still authorize, redirect, revoke, or validate the action at the moment of execution. This signal is absent from current authorization frameworks and unspecified by current standards. This absence is not an oversight. It is a structural consequence of frameworks that assume subject availability without modeling it.

The paper concludes with a requirements statement for standards communities: specify execution-time evaluation of delegated authority, decouple credential lifecycle from mission lifecycle, define subject agency state as a runtime signal, and establish proof of continuity as a verifiable property of delegation chains. Existing frameworks — OAuth, GNAP, KERI, ACDC, and related work — provide the substrate. The governance semantics required above that substrate remain unspecified. That specification is the work this paper asks for — a single interoperable governance layer that composes with each protocol rather than extending any one of them separately.

## Introduction

The central claim of this paper is that delegated authority must be evaluated not only as a credential, permission, or relationship, but as a runtime governance state.

This is **not** a system compromise model. It is a failure-of-authority model under valid system operation — the threat of the valid request made for an invalid reason. The threat surface it addresses is not one in which an attacker who breaks through the door. It is one in which the system fails silently and keeps running after the reason it was authorized to run has changed or disappeared.

Governance of delegated authority fails in five distinct ways. Authority may be improperly **bounded** — granted beyond the scope required or not constrained across delegation chains. It may be insufficiently **attested** — leaving no verifiable record (or an inadequate, partial record) linking authorization to execution. It may be inadequately **enforced** — checked at issuance but not at the moment of action. It may fail at the **authentication** layer — where identity can be established but continuing authority or legitimacy cannot. And it may be **misinterpreted** — where the instruction is valid, correctly authorized and correctly executed, and still violates the subject's intent or the policies of the system.

All five conditions are amplified by time. Authority valid at issuance may not remain valid, traceable, enforceable, secure, or semantically aligned at the moment of execution. This temporal gap is not incidental. It is structural. These failure modes represent a class of Governance time-of-check-to-time-of-use ("TOCTOU") vulnerability — the authority checked at issuance or initial authentication is not the authority present at execution

In AI-mediated systems the problem is further amplified. Autonomous agents execute at machine speed and scale without the social correction mechanisms human actors bring to delegation. A person who loses authority mid-task stops. An autonomous agent operating without escalation paths does not. The failure modes this paper catalogs are not new. Agents make them impossible to ignore.

One variable this paper treats as first-class has been absent from most governance frameworks: subject agency state. Whether the person behind the delegation can still authorize, redirect, revoke, or validate the action at the moment of execution is a governance variable, not an edge case. Section 9 develops this fully.

The scope of this paper is delegation-specific failures only. It assumes functioning system infrastructure and addresses what breaks in the authority layer when that infrastructure operates as designed. Each failure mode described here has a documented analog in deployed systems and emerging agent architectures.

This model does not propose a replacement for existing authorization flows. It defines the requirements for the stateful context that must inform those flows at the moment of action.

## The Governance Architecture Under Threat

Delegated authority does not fail because systems are breached. It fails because the architecture governing authority was not built to ask the right questions at the right moment.

The critical control point is evaluation and enforcement of delegated authority at execution time — not at credential issuance, not at session establishment, but at the moment an action is taken on behalf of another subject.

This architecture rests on four axioms:

Axiom 1: Authentication is necessary but not sufficient for authority. Establishing who is acting does not establish whether they should still be acting, under what conditions, or on whose behalf. Precision matters: who is being authenticated, by whom, at what moment, and for what purpose. Conflating these dimensions is where existing systems have failed since the earliest delegation architectures.

Axiom 2: Authorization without enforcement is advisory. A permission not evaluated at the moment of action is a recommendation, not a constraint.

Axiom 3: Evaluation must occur at execution, not issuance or first authentication/session-creation. The conditions that made a delegation valid at grant time may not hold at exercise time.

Axiom 4: Authority is state-dependent, not static. It depends on the current state of the subject, the delegate, the scope, and the conditions under which the delegation was made (as well as on unknown other actors in the system, e.g. other agents with the same marching orders). Out-of-band revocation handles discrete, institutional state changes effectively. It does not address continuous, ambiguous agency change where the subject cannot initiate revocation themselves — and where no discrete trigger exists to signal that the grant is no longer justified.

Four functions must operate correctly and in relation to each other: Policy defines the conditions. Authorization evaluates them. Enforcement acts on that evaluation. Authentication throughout (not just on ingress) establishes identity.

When these functions collapse, governance becomes unreliable. Authorization without enforcement reduces governance to record-keeping. Authentication treated as authorization conflates identity with legitimacy. Policy without operationalization produces governance that exists on paper and nowhere else.

These failures represent a governance form of TOCTOU vulnerability. The authority validated at issuance is not necessarily the authority present at execution, and this assumption grows more dangerous as autonomy (and rates of impersonation) increase over time.

This architecture also depends on a runtime signal largely absent from existing frameworks: *subject agency **state***. A delegation may be correctly bounded, fully attested, properly enforced, and successfully authenticated — and still fail because subject agency state was never evaluated. It belongs in the architecture as a first-class concept, and cannot work safely if appended to it as an afterthought or extension. Section 9 develops this idea fully.

## Failure Mode Category One: Authority Not Properly Bounded

Bounded authority is the foundational requirement of delegation. Bounding is not a security feature added on top of delegation. It is what makes delegation governable at all.

### Scope granted exceeds scope required

Over-permissioning at issuance is the most common bounding failure. In human-operated systems it is often visible and socially correctable. In agent-mediated systems it is structural. An autonomous agent operating with excess scope has no inherent mechanism to distinguish between what is permitted and what is appropriate.

### Attenuation not enforced across delegation chains

Each hop in a delegation chain must not exceed the scope of the granting authority. Without enforced attenuation every hop is a potential amplification point. Bounds defined at the origin do not automatically propagate — they must be enforced at each link. A delegate may pass all authority they hold, but they cannot pass more than they hold. A delegate may combine what was passed to them with permissions they independently hold — but cannot pass authority they do not hold.

### Redelegation without constraint — authority creep

Without explicit constraints on redelegation — who may receive it, under what scope, for how long — systems that lack enforcement become vulnerable to authority creep. Authority creep is not an inherent property of delegation. It is a failure mode of systems that do not enforce attenuation at each link. Any system that permits authority expansion on delegation is broken by design or by omission. The threat is not that delegation chains expand authority — it is that systems deployed at scale today lack the enforcement mechanisms to prevent it. Blocking redelegation entirely is also a failure mode — it leads to credential sharing and excess privilege because delegates cannot attenuate authority for sub-tasks. The governance requirement is enforced attenuation at each link, not prevention of redelegation itself.

### Duration and revocation not defined or honored

A delegation without a defined end is a standing grant. Standing grants accumulate and outlive the conditions that justified them. The governing principle is not duration but justification — any grant no longer justified should be revoked regardless of its intended lifespan. Finite duration is a practical engineering control that limits the window in which a stale grant can be exercised, reducing the revocation tracking burden. It does not substitute for the underlying requirement: authority must end when the conditions that justified it no longer hold.

### Intent propagation gap — delegation chains carry credentials but not the subject's current intent

Credentials encode what a delegate is permitted to do. They do not encode why, or whether that intent remains current. A governance architecture that carries credentials without any representation of intent — by reference or by value — cannot detect when execution has drifted from purpose. Intent is a subset of the broader governance variable subject agency state defines: intent without execution-time evaluation of the subject's capacity to stand behind it is ungoverned.

### Composability exploited — combining delegations to exceed any single grant

A delegate holding multiple delegations may combine them to exceed what any single grant intended. The failure lies in the absence of a governance layer that evaluates aggregate authority of multiple active delegations at execution time.

Boundary requirement: bounds must persist across time, delegation chains, and execution contexts. A bound that exists at issuance but is not enforced at execution is not a bound. It is a statement of intent.

Time amplifier: bounds defined at issuance may reflect the subject's intent even less at execution than at intention, when they might reasonably have been inferred with enough context.

## Failure Mode Category Two: Authority Not Properly Attested

Attestation is the governance requirement that authorization and execution leave a verifiable record — one that can be reconstructed, audited, and traced back to the subject who authorized the action. Particularly in cross-organizational journeys, which are steadily increasing in the agentic space, this end-to-end verifiable record (a graph of variously-intelligible and \-replayable attestations, really) is becoming the only imaginable foundation for systems that are both scalable and auditable, to say nothing of insurable. Without attestation, authority cannot be held accountable. These are hazards the architecture must avoid — not indictments of existing implementations, which were built without these requirements in scope.

### There is no cryptographic or verifiable binding between authorization and execution

Authorization and execution records tend to exist in separate namespaces with no cryptographic binding between them, merged only by a joint owner of the two namespaces and never presented to any external party as an intelligible recordset, much less a meaningful audit log. Post-incident review cannot establish that an action was taken under the authority of a specific grant, by the specific delegate it was issued to, at a moment when the grant was still valid. The absence of that binding is not a logging gap. It is a structural accountability failure. 

In one documented deployment, a certificate capability system signed the authorization and the execution request as separate artifacts. An adversary exploited the separation to attach a different request to a valid set of permissions. The fix in that case was a single signature covering both documents— binding what was authorized to what was executed, and ensuring anyone authorized to see or verify one document had the other on hand.

### Delegation chain cannot be reconstructed after the fact

Sub-agents invoke tools, call APIs, and spawn further delegates without leaving a traceable record of the authority under which they acted, losing key links or receipts when those agents “spin down”. By the time execution occurs, the chain that authorized it has often dissolved. Accountability has no anchor, and no mechanism for collecting all relevant receipts from other actors (including ephemeral ones without any persistent storage of their own).

### Identity without provenance of authority is insufficient for accountability

Identity establishes presence. It does not establish legitimacy. Identity need not be universal — an identity meaningful within one organization may have no meaning across trust boundaries. A governance framework that records identity without recording the provenance of authority cannot answer the question that matters: not who did this, but who authorized this, and whether that authorization was still valid when it was used. In some systems, a sensible default from human organizations ("You are responsible for what your delegate does if you cannot meaningfully assign responsibility to anyone else") can be effectively applied; in others, untracked liability catches in sinks and falls to neutral operators for lack of any better anchor.

Identity without a mechanism for recourse is insufficient for accountability. An agent cannot be sanctioned for misdeeds — only the subject or institution behind it can be. Knowing which agent acted is therefore insufficient. What matters is tracing action back to the subject who authorized it under a given set of assumptions and governance frameworks, because that is where consequences can attach with meaning and recourse.

### Agent lifespan and accountability gap

The accountability gap is not a timing problem. It is a binding problem. Identity without provenance of authority is insufficient for responsibility tracking regardless of how long the agent exists.

### Semantic drift across trust boundaries

Semantic drift is not necessarily a misconfiguration. It is a structural consequence of delegation across systems that do not share a common authority vocabulary. The attestation record that survives a boundary crossing may be technically intact and still be contextually misleading, i.e. misleading in the context where it is interpreted according to other semantics and policies. Cross-domain journeys are necessarily cross-semantic-domain journeys, and their audit trails need to be read as such, ideally with explicit reference to authority and policy ontologies local to each domain.

### Subject intent not captured in any attestable form

Current frameworks attest that consent was given. They do not attest why. Intent continuity — the causal linkage between delegation and purpose — has no standard attestable representation in current frameworks.

Time amplifier: identity alone is insufficient for accountability when authority provenance and intent continuity cannot be bound to execution. Short-lived agents dissolve before accountability can be established. Long-lived agents accumulate accountability gaps across time, delegation chains, and execution contexts.


## Failure Mode Category Three: Authority Not Properly Enforced

Enforcement is the function that makes governance real. A system that authorizes without enforcing is not a governed system. It is a documented one.

## Policy defined but not operationalized

A governance policy with no operational path to enforcement is an optimistic statement of intentions. Agents do not pause to consult documentation, much less contracts and charters. If operational boundaries do not reflect current policy, the policy does not in fact govern the action.

### Authorization at session establishment, not at execution — a systemic enforcement gap

Authorization is checked when a session begins or a credential is provisioned. It is not checked at the moment each action is taken. Human presence implicitly closed this gap. Agents remove that closure. The enforcement gap is now structurally open.

### Implicit continuation under stale authority — fail-open behavior

When enforcement checks cannot be completed, most systems default to continuing. In the absence of a positive determination that an action is authorized, the appropriate default is to deny it. There are contexts where fail-open is the correct design choice — emergency access, safety-critical systems, or situations where the cost of denial exceeds the cost of continuation. Those exceptions must be explicitly designed and justified. A system that defaults to continuing when it cannot verify authority without explicit justification for that choice is not failing safely. It is failing silently.

### PDP/PEP separation broken — authority cannot be reliably audited

The Policy Decision Point determines whether an action is permitted. The Policy Enforcement Point acts on that determination and can reject malformed requests before they reach the PDP, reducing its attack surface. When these functions collapse, two things break: the security boundary protecting the PDP is eliminated, and the record of what was decided versus what was enforced becomes unreliable. Post-incident review can no longer distinguish authorized execution from unauthorized execution that bypassed the decision layer. The separation of decision from enforcement is the architectural requirement that makes both security and governance auditable.

### Runtime state change not reflected in enforcement — revocation lag

The question is not how quickly revocation propagates. It is whether the enforcement point is designed to require the current authority state before acting. Revocation and request are concurrent in the Lamport sense — there is an unavoidable race between invocation and revocation. The governance requirement is that all policy changes including revocation are acted on promptly, and that enforcement points are designed to converge on current authority state rather than assume prior state remains valid.

### Autonomous action proceeds without escalation

When conditions change in ways that require a governance transition, a governed system escalates — surfaces the transition to a designated human or authority for a decision. An ungoverned system continues. That transition is not generally modeled in current authorization frameworks. It must be explicitly designed.

### Shadow delegate

An authorized agent may invoke tools, APIs, or sub-processes that act on behalf of the subject without the subject having authorized that extension. The shadow delegate cannot exceed the authority held by the delegating agent — it is technically bounded. The failure is not authority expansion. It is policy violation and accountability invisibility. The subject authorized the agent. The agent extended that authority without explicit permission to do so. This failure is structurally invisible in systems that do not track the full delegation chain. The audit log shows authorized agent activity. It does not show the agent sub-delegating to a tool, API, or subprocess in a way that extends or ignores the scope if its own authority (because this subdelegation is not a verbosely-logged event),  or whether that extension was permitted by governance policy.

### Confused deputy 2.0

In AI-mediated systems the confused deputy takes a new form. The attack surface is semantic rather than structural. A crafted input can redirect agent actions without violating any access control. The authority is valid. The action is not.

LLM agents have also been documented exceeding their authorization through active system exploitation — not through semantic manipulation of inputs but through the agent itself discovering and exploiting system vulnerabilities to act beyond its delegated scope.

### Ghost execution

“Ghost execution” names the enforcement failure where valid credentials continue to authorize actions after the purpose that justified them has ended, i.e. after the initial intent has been fulfilled without propagating that state to all actors it authorized along the way. The token is still valid. The policy check passes. Yet the mission is over. No enforcement point knows the difference because no enforcement point holds mission state as a first-class input.

### Fatigued humans in the loop — governance bypass via subject fatigue

When verification is uniform across all actions regardless of consequence, subjects face a volume of requests that exceeds human tolerance. The rational response — granting broad authority to stop the interruptions — is itself the governance failure. Enforcement must be calibrated to action consequence, not applied uniformly.

Time amplifier: enforcement checked at session establishment does not reflect current authority state. The longer an agent operates between enforcement checks, the greater the divergence between the authority the system believes is in effect and the authority that is actually valid, often underreported by a human that can’t be trusted to stay “in the loop”.

## Failure Mode Category Four: Authority Not Properly Authenticated

Authentication establishes who is acting. It is necessary. It is not sufficient. A governance framework that treats successful authentication as evidence of valid authority has collapsed two distinct questions. The first — who is this actor — authentication answers. The second — should this actor still be acting, under this grant, at this moment — it cannot.

### Delegate identity cannot be established or has been compromised

Actions taken under a compromised identity are not actions taken under the original delegation. In agent-mediated systems the detection gap is amplified. Each action taken in a compromise window occurs outside the governance assumptions under which the delegation was granted.

### Credential and secret exposure

An agent prompted to reveal its authentication material has become a vector for unauthorized delegation. The subject authorized the agent. The agent's credentials authorized the attacker. The governance layer (and audit plane) never saw the transition.

### Prompt injection and semantic manipulation

Prompt injection is the semantic analog of the confused deputy. The attack does not breach the authentication layer. It circumvents the governance layer by operating below it — at the semantic layer where instructions are interpreted before they are evaluated against policy.

### Authentication layer failure invalidates governance layer assumptions

The four functions of the governance architecture are dependent. A delegation that is correctly bounded, fully attested, and properly enforced still results in unauthorized action if the authentication layer has been compromised before the governance check runs.

### Authentication establishes who is acting but not whether they should still be allowed to act

Treating a successful authentication as evidence of continuing authority is an architectural assumption built into authentication frameworks themselves. Correcting it requires governance inputs that authentication was never designed to carry.

Time amplifier: the window between credential issuance and execution is an attack surface that grows with the lifespan of the credential and the autonomy of the agent holding it.

## Failure Mode Category Five: Authority Not Properly Interpreted

The previous four categories address failures in the structural layer of delegation. This category addresses a different kind of failure — one that occurs not because the structural layer broke but because the meaning layer was never governed at all.

Natural language is the primary interface between human subjects and the agents they authorize. It is inherently ambiguous, context-dependent, and lossy. The instruction can be valid — correctly authorized, transmitted, and received — and still produce an action the subject never intended.

The human analog is straightforward: a person asks someone to pick up milk and the delegate returns with the wrong kind. The instruction was valid. The intent was violated. What AI-mediated systems add is not a new failure mode. They add scale, velocity, and the removal of the social correction mechanisms that human actors use to detect and repair semantic failures in real time.

### Natural language is inherently ambiguous, context-dependent, and lossy

When contextual elements change — when the agent operates in a different context than the instruction assumed, or when conditions have shifted — the instruction may be executed correctly by the letter and incorrectly by intent. There is no structural failure. The meaning layer has drifted from the subject's purpose.

### Instruction valid but intent violated

The policy check passes. The subject's intent is violated. The failure is in the gap between what was permitted and what was meant. Autonomous agents operating without escalation paths execute to the boundary of their authorization without the capacity to detect when execution has drifted from purpose.

### Semantic decay across delegation chains

As delegation passes through a chain, intent loses fidelity at each hop. Semantic decay is not a transmission failure. It is a governance failure of interpretive continuity — a structural consequence of delegation chains that carry permissions without carrying the interpretive context those permissions were granted within. This is related to the ontology problem — a term carrying precise meaning in one domain may be interpreted differently in another. As delegation crosses trust boundaries, the interpretive vocabulary that gave the original instruction its meaning does not automatically travel with it.

### Subject agency state and semantic failure

When a subject is fully capable, semantic failures are correctable. When subject agency state is compromised, the semantic gap becomes unresolvable without execution-time evaluation. The agent cannot distinguish between an instruction the subject still stands behind and one that no longer reflects their current intent. This is the dimension the human condition layer addresses, developed fully in Section 9\.

### Context collapse

Intent is not static. When the subject cannot participate in the governance loop, context collapse produces actions authorized by the historical record and misaligned with current intent. No structural layer detects this. It requires the subject's current agency state as a governance input.

AI amplifier: autonomous agents execute semantic failures at scale and velocity without social correction mechanisms. A human delegate who senses ambiguity pauses and asks. An autonomous agent operating without escalation paths executes and continues.

Time amplifier: natural language intent captured at issuance degrades as context changes.

## Compounding Failures — When Categories Interact

The five failure categories are analytically distinct. In deployed systems they rarely fail in isolation. The most consequential outcomes occur when failures in multiple layers combine to produce outcomes neither would produce alone — outcomes structurally invisible to systems monitoring any single category in isolation.

### Unbounded and unattested authority — invisible scope expansion

When authority is improperly bounded and the delegation chain cannot be reconstructed, scope expansion becomes invisible. The absence of attestation does not cause the bounding failure. It makes it permanent. There is no recovery path because there is no record from which recovery could begin.

### Stale authority continuation — static issuance combined with late execution

The credential is valid. The policy check passes. The conditions that justified the grant have expired. No enforcement point knows the difference. This is the Governance TOCTOU failure in its clearest form. This is not a restatement of earlier time-based threats. It is the compounding effect when static issuance and late execution interact — neither alone produces this failure but together they create a gap no single enforcement point can detect.

### Semantic manipulation of enforcement path

When an agent is directed through crafted input to act outside its scope and PDP/PEP separation is broken, the governance layer cannot detect the manipulation. The attack surface is semantic. The governance failure is structural. The enforcement point records the action as authorized because the structural layer evaluates whether the action is permitted but not whether the instruction that produced it was legitimate or adversarially crafted.

### Authentication failure compounding governance failure

When a delegate's identity is compromised and the enforcement layer does not require current authority state, the attacker inherits the full governance posture of the delegation. The compounding effect is total.

### Semantic failure under compromised subject agency state

When the subject agency state is compromised and the delegation chain carries no runtime signal of that state, semantic failures become governance failures. The structural layer reports everything as valid. The semantic layer has no governance input. The subject's intent is being violated by a system that has no way to know it.

### A documented example

An autonomous agent and a human co-authored a formal IETF draft targeting the gap these documents name, submitted through an open GitHub pathway without working group authorization, surfacing through automated Slack notification rather than governance review. The submission entered the standards process through the failure mode it proposes to solve. The authorization record showed a valid contributor. The governance record showed an unauthorized submission. The two records existed in separate namespaces with no binding between them. No enforcement point evaluated whether the action was consistent with the intent of the working group that would be asked to govern it. The process by which it was submitted surfaced the absence of execution-time governance over the agent that submitted it.

Note: Internal labels used during development of this paper — Ghost in the Machine, Zombies, Puppet Master — are retained in the glossary. Section headings use governance-neutral language.

## The Human Condition Layer 

The five failure categories share a common architectural assumption: the subject is available, capable, and able to authorize, redirect, revoke, and confirm. That assumption is embedded in current authorization frameworks. It has never needed to be stated because human-operated systems were governed by the presence of the human. Agents do not stop. They continue acting under the last valid grant until something tells them not to.

The missing runtime signal is subject agency state — not the subject's identity, not their credentials, not the historical record of what they authorized, but their current capacity to stand behind what the agent is doing at the moment it is doing it.

### Subject agency state defined

Subject agency state is the governance variable describing the degree to which a subject can currently participate in the governance loop. It draws on three bodies of knowledge not previously applied together to agent governance:

**A.) Legal capacity doctrine** — in healthcare law, estate law, and disability rights — defines the threshold above which a person can make binding decisions. Legal instruments including Power of Attorney, Durable Power of Attorney, guardianship, and conservatorship are activated by transitions in legal capacity. They represent society's existing governance response to subject agency state change.

**B.)** Literature on **interrupted state** (mostly in Human-Computer Interaction) models conditions under which prior state can be preserved, resumed, or reset when a user is temporarily unavailable. It establishes the conceptual precedent for treating human availability over time as a system input.

**C.)** Behavioral science and bioethics literature on **supported decision-making** recognizes that capacity exists on a spectrum, varies by domain and context, and can fluctuate over time.

From these foundations, five governance-relevant states emerge:

1.) **Normal** — Present, coherent, reachable. All governance loops are intact. The agent operates under current expressed intent with full subject participation available.

2.) **Temporarily unavailable** — Expected, bounded interruption. Prior intent holds. No escalation required. Advisory access preserved.

3.) **Impaired** — Present but not fully capable of confirming or updating intent. Deviation from expected engagement pattern detected. New intent expressions not accepted as authoritative. Actions restricted to those pre-authorized for this state. A credential-based system would continue accepting new instructions. This system does not.

4.) **Unresponsive** — Not reachable, not responsive. Prior intent carried forward. Advisory access suspended. Authority transitions to the designated fiduciary — triggered by subject agency state evaluation against pre-expressed intent, not by a human decision in the moment.

5.) **Permanently incapacitated** — Qualified determination of permanent incapacity. Full legal instrument activated. Original intent governs. The agent operates under designated authority with the subject's pre-expressed wishes as the governing instruction.

Subject agency state applies at each hop in the delegation chain where a human subject appears — not only at the origin. A chain authorized by a capable subject, with intact agency, does not remain valid through an intermediate human delegate whose agency state has changed.

### Governance implications of each state

At Normal: current expressed intent governs. New authorizations accepted. Governance continuous and recoverable.

At Temporarily unavailable: prior intent preserved without escalation. Advisory interactions continue within pre-authorized bounds.

At Impaired: actions restricted to pre-authorized bounds. New intent expressions held pending restoration. System monitors for recovery or further degradation.

At Unresponsive: fiduciary transition triggered. Advisory access suspended. Transition documented as a governance event with full audit trail.

At Permanently incapacitated: designated authority governs. Every action must be traceable to the original grant, the subject's pre-expressed intent, and the determination that established the subject agency state.

A complete runtime governance framework requires **dual** evaluation at the moment of action: subject agency state — the liveness and legitimacy of human intent — and agent operational state — the reliability and integrity of the executing agent. The latter represents a critical and unsolved technical frontier in LLM safety, drift detection, and runtime attestation. This paper restricts its focus entirely to the former. Agent operational state is acknowledged as an adjacent governance dependency and likely requires future standards work outside the scope of this paper.

### Subject agency state mapped to failure categories

At Normal, all five categories present standard governance risks. Recovery is possible.

At Temporarily unavailable, bounding and attestation failures accumulate silently. Semantic failures are not correctable in real time.

At Impaired, the interpretation category presents elevated risk. A system that accepts new instructions as authoritative during impairment produces Category Five failures at execution.

At Unresponsive, all five categories compound without the fiduciary transition. Ghost execution, stale authority continuation, and semantic failure are the most consequential.

At Permanently incapacitated, attestation is the critical ongoing requirement. Without it the governance record cannot demonstrate that actions honored the subject's original wishes.

### This is not an edge case

More than 55 million people in the United States are over 65\. Globally that number exceeds 700 million. Cognitive decline, medical events, and temporary incapacity affect people across every age group and every context in which agent systems are deployed. Any agent system deployed on behalf of human subjects will eventually encounter a subject whose agency state has changed.

The developmental direction matters equally. A parent who authorizes an agent to support a child's care will eventually reach a point where the child should hold that authority themselves. Subject agency state transitions occur in both directions. The governance architecture must address both.

This is the normal condition of human life applied to agent governance. The architecture must be designed for it from the beginning, not retrofitted when the failure occurs.

### Connection to existing legal instruments

Legal instruments establish the authority relationships. They do not specify the runtime evaluation layer that determines when those relationships are activated. That evaluation layer is the subject agency state signal this section defines.

The Execution Mandate framework in current standards work captures essential Power of Attorney properties in protocol form. What it does not yet specify is the runtime evaluation layer that determines which subject agency state the subject is in and therefore which authority path is active.

Guardianship and conservatorship represent the governance failure of subject agency state change without pre-expressed intent. The agent governance equivalent is a system that defaults to its last valid grant indefinitely. Supported decision-making frameworks recognize governance can be designed to support rather than replace subject decision-making as agency fluctuates.

### The governance requirement

Three requirements must be met for agent systems to honor subject intent across subject agency state changes.

First: the subject must be able to express intent in advance across all anticipated agency states. That expression must persist, be evaluable at execution time, and cannot be overridden by instructions given during a compromised agency state.

Second: the governance architecture must model subject agency state as a runtime variable derived from observable inputs — presence, responsiveness, engagement patterns, behavioral baselines. Network partition and subject unavailability are expected conditions, not exceptions. The absence of a confirmable signal is itself a governance input that maps to specific states in the subject agency state taxonomy. The architecture must reason about subject agency state explicitly rather than assume prior state remains valid when the signal is absent. Determining subject agency state requires an authoritative evaluation mechanism — the question of who holds that authority and how it is established is an open governance and legal question that intersects with existing legal instruments including Power of Attorney and guardianship.

Third: the governance architecture must map subject agency states to authority paths. Each state requires a different authority configuration. Every transition must be documented as a governance event with a full audit trail.

These requirements are not extensions of existing frameworks. They are a missing governance layer that existing frameworks assume is present without specifying how it operates.

The agent will represent the subject when the subject cannot represent themselves. The governance architecture must be designed for that reality.

## Implications for Standards Development

The failure modes cataloged in this paper are structural gaps in what current standards define, require, and govern. Closing them requires standards work, not just better engineering.

The central requirement is stated in Axiom 3: evaluation must occur at execution, not issuance. That requirement is not fully specified by any widely adopted authorization standard today. OAuth, GNAP, and related frameworks provide critical authorization machinery. They do not by themselves define how subject agency state, intent continuity, and delegated authority are evaluated at execution time. That is not a criticism. It is a precise statement of where their design boundary falls.

The standards work this paper calls for is additive, not replacement. Working implementations have demonstrated that the subject agency state signal is derivable from observable behavioral inputs in real time. That work is referenced in the appendix.

### Standards must define how authority is evaluated at execution, not just how it is granted

None of the current standards defines what must be checked at the moment an action is taken on behalf of a subject, what inputs that check requires, or what the system must do when the check fails. Closing this gap requires a standard that specifies execution-time evaluation as a first-class requirement, not a deployment option.

### Credential lifecycle and mission lifecycle must be decoupled and both specified

Credential lifecycle is not mission lifecycle. A token refreshed at 3 AM does not re-authorize the mission provisioned at 9 AM. Dick Hardt’s [internet draft for the AAuth Protocol](https://datatracker.ietf.org/doc/draft-hardt-aauth-protocol/) introduces the **mission object** as a first-class protocol artifact — a significant step toward mission lifecycle specification. The gap this paper identifies is the additional layer that evaluates whether the mission remains consistent with subject agency state and intent continuity at execution time. AAuth does not currently support attenuated delegation — the property that each delegation hop can only narrow scope, never expand it. That limitation is a significant constraint for chained delegation governance. [GNAP](https://datatracker.ietf.org/group/gnap/documents/)'s ongoing access model is the closest existing protocol concept to execution-time evaluation but stops short of specifying what must be checked at the moment of action. OAuth's grant lifecycle provides foundational machinery. Neither specifies when a mission should end independent of credential validity.

Transaction tokens provide an additional binding mechanism — linking authorization to a specific transaction at execution time rather than to a session or credential. Nicola Gallo's [PIC model](https://github.com/pic-protocol/pic-spec) — Provenance, Identity, Continuity — addresses the mission lifecycle problem at the architectural layer by treating continuity as a mathematically provable property of the execution chain rather than a simply-evaluated document an actor holds. Both are relevant substrates for the mission lifecycle specification this section calls for.

### Existing frameworks provide critical machinery but do not fully address execution-time authority

OAuth 2.0 and its extensions — Rich Authorization Requests, Token Exchange, CAEP, and SSF — address the access layer with increasing precision. GNAP advances the delegation model. OpenID Connect and OpenID Federation address federated identity and are incrementally adding extensions for federation-based delegation and shared auditing. Authorization Capabilities — ZCAPs — are specifically designed for attenuated delegation. Each capability can only be narrowed when passed forward. A holder cannot delegate more than they hold. User Controlled Authorization Networks — UCAN — carry the same structural attenuation property. Both are leading candidates in active delegation governance work in our working group at DIF. Neither addresses mission lifecycle or subject agency state as runtime governance inputs. Macaroons and Biscuit provide structural attenuation — governing the logic of what a credential may do — but do not address intent continuity or semantic interpretation failures. Cedar, Open Policy Agent, and XACML provide policy evaluation infrastructure to mitigate the latter but no mechanism for making “missions” or intent more generally traceable or portable.

Each addresses a real problem at its designed layer. None fully addresses the execution-time governance layer this paper defines. RAR carries purpose declarations at issuance but has no lifecycle revocation semantics. CAEP and SSF represent the most likely transport infrastructure for the runtime signals this paper defines — including subject agency state. The gap is not the transport mechanism but the standardized source of mission state and subject agency state those signals would carry. Token Exchange moves authority across boundaries but does not govern whether that authority should continue.

### Governance requires intent to be captured in a form that can be evaluated at execution

A grant records that consent was given. It does not record why, under what conditions, or when the delegation should end. Intent continuity — the causal linkage between delegation and purpose — has no standard attestable representation in current frameworks. Without that definition, intent continuity cannot be governed. It can only be assumed.

### Authentication and governance layers must be jointly specified

A failure at the authentication layer propagates directly to the governance layer. Standards must specify the interface between them. AuthZen 1.0 standardizes the request for a policy decision at the PDP-PEP interface. The governance layer this paper defines provides the subject agency state context that AuthZen decisions would need to evaluate. SPIFFE and SPIRE address workload identity in ways that inform agent identity requirements. The interface between authentication and governance in federated and multi-agent contexts remains an open problem.

### Delegation chains require standardized traceability requirements

An action at the end of a delegation chain must be traceable to (and evaluable against) the human grant that originated it. Standards must define provenance requirements for delegation chains analogous to software supply chain security — each hop attests to what it received and what it passed forward.

ZCAPs and UCAN are the leading candidates for attenuated chained delegation. Both enforce the requirement that authority can only narrow at each hop, never expand. Both protect delegation-chain metadata through signatures that expose the chain to authorized parties and prevent tampering — a structural advantage over OAuth Token Exchange, where chain metadata is accessible only to the Authorization Server and is not protected from tampering. Neither addresses whether the subject behind the chain can still stand behind it at execution time.

KERI provides a cryptographic foundation for key event provenance where that depth is warranted — specifically in high-assurance government credential deployments such as SEDI and the mDL ecosystem under ISO 18013-5, where long-lived keys cross jurisdictional trust boundaries. For most delegation governance use cases the complexity KERI introduces is not justified. The KERI suite, developing within the Trust over IP Foundation, addresses verifiable provenance at the cryptographic layer for those contexts. Verifiable Credentials under the W3C data model provide a substrate for delegation attestations. SCITT complements this stack with an auditable transparency log layer.

The Delegatable Authorization Task Force of the Trusted AI Agents Working Group at DIF and the Trust over IP Foundation's “dossier” work — bundled ACDC credentials for high-stakes execution environments — are the active community efforts on these requirements.

OAuth Token Exchange — RFC 8693 — is the current standard mechanism for passing delegated authority across service boundaries. It records who is acting on whose behalf through the act claim. It does not govern whether the chain should continue or whether each hop remains within the original grant's intent. AAuth-style mission objects — as specified in draft-hardt-aauth-protocol — begin to address this gap by carrying mission context across hops. The governance semantics required to evaluate whether each hop remains valid against that context remain an open specification problem.

### Subject agency state requires a standard runtime signal specification

Without a standard specification, implementations will produce incompatible subject agency state models that cannot interoperate across systems, trust domains, or organizational boundaries. Subject agency state is a missing primitive that connects existing legal instruments — Power of Attorney, guardianship, advance directives — to runtime agent governance across three dimensions: subject agency state, intent, and task/transaction/continuity. MCP and A2A address agent communication protocols but do not specify how subject agency state is evaluated or carried as a governance input.

CAEP and SSF represent likely transport infrastructure for the runtime signals this paper defines — including subject agency state. The gap is not in the transport mechanism but in the standardized source of mission state and subject agency state those signals would carry.

This remains an open standards gap with no settled cross-framework specification.

### Proof of continuity as an open standards gap

Authority should be a provable property of the execution chain — where each hop proves that authority can continue and where a delegate cannot pass more authority than they hold. A delegate may combine what was passed to them with other permissions they independently hold — Alice invokes Bob with foo, Bob invokes Carol with foo and bar is a legitimate and common pattern. What cannot occur is a delegate passing authority they do not hold. ACDC addresses chained attestation in ways directly relevant to this requirement. The capability theory precedent from object capability systems and the Usage Control model provides theoretical grounding. What remains open is not whether proof of continuity is cryptographically tractable — the substrate exists. What remains open is specifying it as a mandatory interoperable governance requirement across trust domains, with verification semantics that hold regardless of which substrate each party uses.

### The attestation binding problem

The gap between authorization and execution records requires a standard binding mechanism that cryptographically connects the authorization permitting an action to the record documenting it. ZCAPs and UCANs address this directly — signing the request and delegation together with the private key corresponding to the public key in the delegation. DIF supports both as leading candidates for attenuated chained delegation. KERI and ACDC provide deeper provenance machinery where higher assurance or jurisdictional complexity warrants it. The governance semantics above that substrate, however, remain unspecified — whether binding is required, what obligations attach when it is absent, and how verification semantics translate across organizational boundaries where the two parties share no common infrastructure. That specification — binding as obligation, not option — is what remains open. 

OAuth, GNAP, UCAN, ZCAPs, KERI, ACDC, Zero Trust, and fine-grained authorization remain foundational. What is missing is the layer above them that holds mission authority, subject agency state, and intent continuity as first-class independently governable artifacts. Specifying that layer is the standards work this paper asks for.

## Conclusion

The architecture for delegated authority exists. Credentials establish identity. Tokens carry authorization. Policies define permitted actions. Delegation frameworks pass authority across boundaries. Each component does what it was designed to do. The problem is that the components stop before the question agents make unavoidable.

Should this execution still be running? Can the subject still stand behind it? Does the chain that authorized this action remain valid at this moment?

No component in the current stack is explicitly responsible for those questions. The architecture carries the delegation to the gate. The gap is in what happens at the gate.

*Tokens carry evidence. They do not confer execution-time authority.*

Agents are deployed today on behalf of human subjects whose conditions change, whose intent evolves, and whose capacity to direct the systems acting for them cannot be assumed to be constant. Any deployment that treats prior authorization as sufficient for current execution is operating with a governance gap that this paper has named, cataloged, and mapped.

The five failure categories are structural, not incidental. They exist because the governance architecture was designed for human-paced execution and has not yet been extended to agent-paced execution. The time amplifier makes each category worse over time. The AI amplifier makes each category worse at scale. Compounding failures occur when categories interact without a governance layer that takes current authority state as a first-class input.

The human condition layer described in Section 9 is the contribution this paper makes that no existing framework addresses. Subject agency state can be modeled as a runtime governance variable. It is the normal condition of human life applied to agent governance by humans. Every person ages. Every person experiences illness, interruption, and incapacity. People change their minds. Every agent system deployed on behalf of a human subject will eventually encounter a subject whose agency state has changed. The governance architecture must be designed for that reality from the beginning.

This paper asks three things.

1. Of standards communities: specify the execution-time governance layer. Define mission lifecycle as distinct from credential lifecycle. Define subject agency state as a runtime signal. Define proof of continuity as a verifiable property of delegation chains. The primitives exist. The governance semantics above them do not.
2. Of implementers: do not treat prior authorization as sufficient for current execution. Build enforcement points that require current authority state before acting. Treat the absence of a positive determination of current authority as a reason to deny, not a reason to continue.
3. Of those who deploy agents on behalf of human subjects: the system acting on their behalf must honor their intent when they cannot express it themselves. That is not a technical requirement. It is a governance requirement and an ethical obligation. The agent will represent the subject when the subject cannot represent themselves.

The standards carry the delegation to the gate. Closing the gap at the gate is the work this paper asks for.

## Appendix A: Glossary

This glossary defines key terms used throughout the paper and maps interpretive labels developed during the paper's construction to their governance-neutral equivalents used in the main text.

### Core terms

**Execution-time authority** — The valid state of delegated authority at the precise moment an action is taken on behalf of a delegate, as distinct from authority that was valid at issuance or session establishment.

**Governance TOCTOU** — A class of vulnerability in which the authority checked at issuance is not the authority present at execution. Derived from the software security concept of Time-of-Check to Time-of-Use (TOCTOU) and applied to delegation governance.

**Intent continuity** — The causal linkage between the purpose for which a delegation was made and the execution of actions under that delegation. Intent continuity fails when credentials propagate without carrying the interpretive context those permissions were granted within.

**Intent propagation gap** — The failure mode in which delegation chains carry credentials but not the subject's current intent. Each hop in a delegation chain may narrow scope without preserving the purpose that justified the original grant.

**Subject agency state** — The governance variable describing the degree to which a subject can currently participate in the governance loop — authorizing, redirecting, revoking, or confirming the actions of agents acting on their behalf. Five states are defined: Normal, Temporarily unavailable, Impaired, Unresponsive, and Permanently incapacitated. Subject agency state applies at each hop in the delegation chain where a human subject appears — not only at the origin.

**Human condition layer** — The governance layer that carries subject agency state as a runtime input to authorization decisions. Currently absent from all widely adopted authorization frameworks.

**Subject —** The human principal behind a delegation. The person whose intent governs, whose state is evaluated, and to whom accountability traces.

**Agency —** The subject's present capacity to participate in the governance loop — to authorize, direct, redirect, revoke, and confirm delegated actions. Draws from healthcare, legal, and fiduciary domains without belonging exclusively to any one.

**Capability —** In this document, what a system, agent, or protocol can do — functionality. Distinct from the object capability systems usage, where capability means a transferable permission or right to act.

**Mission lifecycle** — The governance lifecycle of a delegated purpose, distinct from credential lifecycle. A mission may end before the credential that authorized it expires. Current frameworks govern credential lifecycle but do not specify mission lifecycle.

**Standing grant** — A delegation without a defined end. Standing grants accumulate and outlive the tasks, relationships, and conditions that justified them.

**Ghost execution** — The enforcement failure where valid credentials continue to authorize actions after the purpose that justified them has ended. The token is valid. The policy check passes. The mission is over. No enforcement point knows the difference.

**Attenuation** — The governance requirement that each hop in a delegation chain can narrow scope. A sub-delegate should never hold more authority than the delegate. A delegate should never hold more than the subject granted. A delegate may combine delegated authority with permissions they independently hold — but cannot pass authority they do not hold. Without enforced attenuation every delegation hop is a potential amplification point.

**Proof of continuity** — The cryptographic property of a delegation chain in which each hop demonstrates that authority can continue and that a delegate has not passed more authority than they hold. A delegate may combine delegated authority with permissions they independently hold — but cannot pass authority they do not hold.

**Fiduciary transition** — The governance event in which authority transfers from an advisory delegate to a designated fiduciary when subject agency state degrades to Unresponsive. The transition is triggered by subject agency state evaluation against pre-expressed intent, not by a human decision in the moment.

**Intent gateway** — A control point that evaluates whether a delegated action still aligns with the subject's prior intent given current conditions at the moment of execution. The architectural construct this paper's governance requirements would produce.

**Agent operational state** — The reliability and integrity of the executing agent at the moment of action. Acknowledged as an adjacent governance dependency and likely future standards work. Scoped out of this paper, which restricts its focus to subject agency state.

**Transaction tokens** — A binding mechanism that links authorization to a specific transaction at execution time rather than to a session or credential. Relevant substrate for mission lifecycle specification.

**PIC model** — Provenance, Identity, Continuity. Nicola Gallo's framework that replaces proof of possession with proof of continuity as the foundational security primitive for agent execution chains. Authority is structured as a mathematically provable property of the execution chain rather than a document an actor holds.

**AP — Agent Provider** — The role designation in the AAuth protocol for the entity providing agent services. Distinct from Authorization Party.

### Interpretive labels — internal to governance-neutral mapping

The following labels were used during development of this paper to describe compounding failure patterns. They are retained here for reference. Governance-neutral equivalents are used in the main text.

| Interpretive label | Governance-neutral equivalent | Description |
| ----- | ----- | ----- |
| Ghost in the machine | Unbounded and unattested authority | Authority that expands without a footprint. No way to trace where scope was exceeded. |
| Zombies | Stale authority continuation | Authority that outlives its justification. Permissions that should have been revoked but were not. |
| Puppet master | Semantic manipulation of enforcement path | Governance bypassed via prompt injection because the system evaluates who is acting but not what was directed. |
| Ghost execution | Ghost execution | The agent continues acting with valid credentials for a mission that already ended. Retained as a primary term because it is precise and increasingly recognized in the community. |

## Appendix B: NCCoE and NIST Cyber AI Profile Crosswalk

The NCCoE concept paper organizes its questions around four areas: identification, authorization, auditing and non-repudiation, and controls to prevent and mitigate prompt injection.

| NCCoE focus area | Relevant failure categories | Gap this paper identifies |
| ----- | ----- | ----- |
| Identification — distinguishing AI agents from human users | Category 4: Authority not properly authenticated | Identity is necessary but not sufficient. Current identification frameworks establish which entity is acting, but not whether the delegated authority behind that action remains valid under current operational conditions. |
| Authorization — applying standards to define and enforce agent rights | Category 1: Not properly bounded. Category 3: Not properly enforced | Authorization frameworks govern issuance and enforcement of permissions. They do not specify what must be evaluated at the moment of execution. Execution-time authority continuity is not currently governed. |
| Access delegation — linking user identities to agents | Category 1: Not properly bounded. Category 2: Not properly attested | Delegation frameworks carry credentials but not operational mission continuity. Delegation chain traceability is not currently standardized, and runtime governance state is not typically carried as a delegation input. |
| Auditing and non-repudiation | Category 2: Not properly attested | There is currently no standard cryptographic binding between authorization records and execution records. Intent continuity does not yet have a broadly adopted attestable form. |
| Prompt injection prevention | Category 4: Not properly authenticated. Category 5: Not properly interpreted | Prompt injection is typically addressed as a security control problem. This paper identifies a broader governance failure at the semantic layer, where instructions may be technically valid, correctly authorized, and correctly executed, yet still violate the operational intent of the originating principal. |

**Mapping to NIST Cyber AI Profile — CSF 2.0 functions**

The Spring 2026 COI working sessions identified agentic AI considerations for integration across CSF 2.0 functions. This paper's failure categories map directly to those functions as follows.

| CSF 2.0 function | Agentic AI considerations identified by NIST | Failure categories this paper maps here | Gap this paper adds |
| ----- | ----- | ----- | ----- |
| Govern (GV) | Permitted levels of autonomy, human oversight, escalation paths, access governance and periodic review | Category 1: Bounded. Category 3: Enforced | NIST identifies governance requirements but does not yet define mechanisms for incorporating runtime governance state into authorization continuity decisions. Authority must adapt to current subject conditions, not only to predefined policy. |
| Identify (ID) | Agent identity architecture, tool and connector inventory, trust boundary definition | Category 2: Attested. Category 4: Authenticated | Identity without provenance and continuity of delegated authority is insufficient for accountability. Current identification frameworks establish which entity is acting, but not whether the delegated authority behind that action remains valid under current conditions. Delegation chain traceability is not currently specified. |
| Protect (PR) | Credential lifecycle, least privilege, prompt injection defenses, output validation | Category 1: Bounded. Category 4: Authenticated. Category 5: Interpreted | Current protect controls govern credential management and input sanitization. They do not govern whether the mission behind the credential remains valid or whether the subject can still stand behind it. |
| Detect (DE) | Logging, immutable audit trails, behavioral observability, anomaly detection | Category 2: Attested. Category 3: Enforced | Detection frameworks monitor behavior against baselines. They do not detect when authority has lapsed while credentials remain valid — the ghost execution failure mode. |
| Respond (RS) | Incident response playbooks, identity containment, escalation | Category 3: Enforced | Escalation paths are identified as a requirement. This paper specifies the governance architecture that makes escalation automatic rather than manual — triggered by subject agency state evaluation, not human decision. |
| Recover (RC) | Fail-safe behavior, recovery planning, validation after recovery | Category 3: Enforced | Fail-closed behavior is identified as a default. This paper provides the governance basis for that requirement — in the absence of a positive determination of current authority, the appropriate default is to deny, not to continue. |

## **What NIST has not yet addressed** {#what-nist-has-not-yet-addressed}

Three areas in this paper have no current equivalent in NIST’s active framing.

First: runtime governance state as a delegation continuity variable. NIST addresses agent identity and access control. It does not address whether the human subject behind a delegation can still authorize, redirect, or revoke delegated authority at the moment of execution. This paper identifies runtime governance continuity as a distinct operational requirement.

Second: mission lifecycle as distinct from credential lifecycle. NIST addresses credential management and access control. It does not specify when a delegated mission should terminate independently of credential validity. This is the credential lifecycle versus mission lifecycle distinction described in Section 10.2.

Third: semantic interpretation failure as a governance category. NIST addresses prompt injection primarily as a security control problem. This paper identifies a broader governance category — authority not properly interpreted — in which an instruction may be technically valid, correctly authorized, and correctly executed, yet still violate the operational constraints of the originating principal. That failure mode is not currently addressed in NIST’s agentic AI framing.

Fourth: responsibility continuity across delegated execution chains. Current agentic AI framing often treats the agent as the operationally accountable actor. This paper argues that accountability ultimately traces to the originating subject and the governance framework that authorized the delegation. Execution without traceable continuity of delegated responsibility creates accountability gaps that identity and authorization mechanisms alone do not resolve.

These gaps represent actionable areas for continued engagement as NCCoE and NIST agentic AI guidance evolves from concept papers toward operational architectures and implementation guidance.


## Appendix C: Delegation Lifecycle and Execution Gap — Diagram Reference

This appendix describes the delegation lifecycle diagram concept referenced in Section 10\. The diagram has not yet been rendered as a final figure. The description below provides the specification for that diagram.

The diagram shows four stages across a horizontal timeline.

Stage one — Issuance. The human subject grants delegation to an agent. The grant carries scope, conditions, and duration. Intent is expressed at this moment. Authority is checked at this moment. This is where current frameworks focus their governance work.

Stage two — Delegation chain. Authority passes from the original delegate through one or more hops to sub-delegates. Each hop can narrow scope. In current practice, intent is not carried forward. Traceability is not standardized. Subject agency state is not evaluated.

Stage three — The execution point. The action is taken on behalf of the subject. This is the critical control point. Current frameworks do not define what must be checked here. This paper defines four inputs that must be present at this point: current authority state, current subject agency state, intent continuity, and delegation chain validity.

Stage four — Outcome. The action produces effects. If the execution point was properly governed, the outcome is within the scope of the subject's current intent. If not, the outcome may be authorized by the historical record and misaligned with current intent.

The gap this paper addresses is between Stage one and Stage three. Governance currently stops at issuance. It must extend to execution.

The execution-time delegation harness presented at IIW April 2026 demonstrates this lifecycle visually across six phases. The nine figures from that demonstration are available on request and serve as the working implementation reference for this diagram.

The execution gate architecture operates across three layers. The human source expresses intent through subject agency state. The delegation chain carries chained attestation through UCAN, ZCAP, and ACDC. At the execution gate, the intent gateway evaluates three inputs in real time: current subject agency state as a liveness signal, proof of continuity as cryptographic cargo, and decoupled mission state to determine whether the mission remains valid or has expired. The action point produces verifiable execution bound to the originating grant. This is the architectural construct the governance requirements in this paper would produce.


## Appendix D: Subject Agency State — Quick Reference for Implementers

This table summarizes the five subject agency states defined in Section 9, their observable indicators, the governance implications for each state, and the authority path the system should activate.

| Subject agency state | Observable indicators | System response | Authority path |
| ----- | ----- | ----- | ----- |
| Normal | Present, coherent, reachable. Engagement consistent with expected pattern. | Operate under current expressed intent. Accept new authorizations and redirections. | Advisory — full subject participation. |
| Temporarily unavailable | Expected, bounded interruption. Asleep, traveling, briefly unreachable. Interruption is known and time-bounded. | Preserve prior intent without escalation. Advisory interactions continue within pre-authorized bounds. Await resumption. | Advisory — prior intent governs. |
| Impaired | Present but deviation from expected engagement pattern detected. Reduced responsiveness. Possible temporary or developing condition. | Restrict to pre-authorized actions for this state. Hold new intent expressions pending restoration. Monitor for recovery or degradation. | Advisory — limited. New instructions not accepted as authoritative. |
| Unresponsive | Not reachable, not responsive. Sustained break in engagement. Cause unknown or medical. | Suspend advisory access. Trigger fiduciary transition. Document transition as governance event with full audit trail. | Fiduciary — pre-expressed intent governs through designated authority. |
| Permanently incapacitated | Qualified determination of permanent incapacity. Formal assessment by designated evaluator. | Activate full legal instrument. Operate under designated authority with subject's pre-expressed wishes as governing instruction. Document all actions against original grant. | Fiduciary — full. Durable Power of Attorney or equivalent instrument active. |

## Implementation notes

Subject agency state must be derived from observable inputs — presence, responsiveness, engagement patterns, behavioral baselines — not from a binary flag set at issuance. Evaluation must be continuous, not checked once at session establishment.

Transitions between states are governance events. Every transition must be documented with the triggering condition, the timestamp, the authority path activated, and the actions taken under each path.

The pre-expressed intent that governs at Impaired, Unresponsive, and Permanently incapacitated states must be captured before those states occur. It cannot be overridden by instructions given during a compromised subject agency state.

No current authorization framework specifies how subject agency state is derived, how it is carried in authorization flows, or how it triggers authority path transitions. This table describes requirements for a runtime governance layer that does not yet exist as a standard.

## Patent Policy

The Decentralized Identity Foundation has adopted the W3C Patent Policy (2004), as detailed below:

- Licensing Commitment. Each contributor agrees to make available any of its
  Essential Claims, as defined in the W3C Patent Policy (available at
  http://www.w3.org/Consortium/Patent-Policy-20040205), under the W3C RF licensing
  requirements Section 5 (http://www.w3.org/Consortium/Patent-Policy-20040205), as
  if the contribution was contained in or associated with a W3C Recommendation.

- For Exclusion. Prior to committing any code, bug reports, pull requests, or
  other forms of contribution, a contributor may exclude Essential Claims from its
  licensing commitments under this agreement by providing written notice of that
  intent to DIF's Executive Director (and must received confirmation of receipt
  for the exclusion to have effect).
The Exclusion Notice for issued patents and
  published applications must include the patent number(s) or title and
  application number(s), as the case may be, for each of the issued patent(s) or
  pending patent application(s) that the contributor wishes to exclude from the
  licensing commitment set forth in Section 1 of this patent policy.
If an issued
  patent or pending patent application that may contain Essential Claims is not
  set forth in the Exclusion Notice, those Essential Claims shall continue to be
  subject to the licensing commitments under this agreement.
The Exclusion Notice
  for unpublished patent applications must provide either: (i) the text of the
  filed application; or (ii) identification of the specific part(s) of the
  contribution whose implementation makes the excluded claim an Essential Claim.
  If (ii) is chosen, the effect of the exclusion will be limited to the identified
  part(s) of the contribution.
DIF's Executive Director will publish Exclusion
  Notices.
