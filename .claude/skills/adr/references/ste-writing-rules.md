# Simplified Technical English Writing Rules for ADRs

## Purpose

Use these rules when writing or editing ADR prose.

The goal is to make the ADR:

* easy to understand for readers under time pressure;
* clear to non-native English speakers;
* precise enough for technical and architectural decisions;
* stable and useful years after the decision;
* concise without removing information required to understand the decision.

These rules apply to prose. They do not require rewriting code, identifiers, API names, class names, protocol names, product names, or established technical terminology.

---

## 1. Use simple sentence structures

Prefer one main idea per sentence.

Avoid sentences that contain several independent decisions, conditions, and exceptions.

**Prefer:**

> The MCP server validates the access token. It then checks the user's domain authorization.

**Avoid:**

> The MCP server validates the access token and, after checking the claims and resolving the user's identity, checks whether the user is authorized to access the requested domain.

Split long sentences when doing so improves clarity.

---

## 2. Prefer active voice

State who or what performs the action.

**Prefer:**

> The MCP server validates the token.

**Avoid:**

> The token is validated by the MCP server.

Use passive voice when the actor is unknown, irrelevant, or intentionally omitted.

---

## 3. Use precise technical subjects

Avoid vague subjects such as:

* this;
* it;
* they;
* the system;
* the solution;
* the approach;
* things;
* stuff.

Name the component, team, interface, or decision explicitly.

**Prefer:**

> The MCP adapter stores the conversation state.

**Avoid:**

> It stores the state.

---

## 4. Prefer common words

Use common English words when they do not reduce technical precision.

**Prefer:**

* use instead of utilize;
* help instead of facilitate;
* start instead of initiate;
* show instead of demonstrate;
* need instead of require, when appropriate;
* about instead of regarding;
* before instead of prior to;
* after instead of subsequent to.

Do not replace established technical terms merely to make them simpler.

For example, keep terms such as:

* authentication;
* authorization;
* Resource Server;
* OAuth;
* PKCE;
* JWKS;
* adapter;
* port;
* idempotency;
* serialization.

---

## 5. Avoid unnecessary nominalizations

Prefer verbs over abstract nouns when possible.

**Prefer:**

> The service validates the token.

**Avoid:**

> The service performs token validation.

**Prefer:**

> The team decided to use OAuth.

**Avoid:**

> The team made a decision regarding the use of OAuth.

Technical terms that are conventionally expressed as nouns may remain nouns.

---

## 6. Avoid unnecessary words

Remove words that do not add meaning.

Common examples:

* in order to → to;
* due to the fact that → because;
* at this point in time → now;
* a number of → several;
* in the event that → if;
* has the ability to → can;
* is able to → can;
* for the purpose of → for.

**Prefer:**

> The server caches the JWKS keys to avoid repeated network calls.

**Avoid:**

> The server caches the JWKS keys for the purpose of avoiding repeated network calls.

---

## 7. Make conditions explicit

Use clear conditional statements.

**Prefer:**

> If the token is expired, the server rejects the request.

**Avoid:**

> An expired token results in the request not being accepted.

For important security or operational conditions, explicitly state:

* condition;
* action;
* consequence.

---

## 8. Keep one tense for stable architecture

Use the present tense for architecture and durable facts.

**Prefer:**

> The MCP server is a Resource Server.

> The application stores conversation state in Snowflake.

Use the past tense for historical decisions or events.

**Prefer:**

> The team rejected the shared client because it did not scale to additional platforms.

Do not switch tense without a reason.

---

## 9. Distinguish decisions from assumptions

Use explicit language.

### Decision

> The MCP server validates MyLogin access tokens locally.

### Assumption

> This decision assumes that MyLogin includes the required claim in the access token.

### Open point

> The exact scope that provides the claim must be confirmed with the AAA team.

Do not present an assumption or unresolved question as a settled architectural decision.

---

## 10. Distinguish facts from rationale

State the decision first, then explain why it was made.

**Prefer:**

> The MCP server uses a dedicated token-validation port. This keeps OAuth token validation separate from the Streamlit authentication flow.

This structure makes the decision easy to find and the rationale easy to understand.

---

## 11. Avoid vague rationale

Avoid phrases such as:

* for flexibility;
* for scalability;
* for better performance;
* for maintainability;
* for security reasons;
* to future-proof the system.

Explain the specific reason.

**Prefer:**

> The server uses a dedicated adapter so the application core remains independent of the MCP protocol.

**Avoid:**

> The server uses a dedicated adapter for flexibility.

---

## 12. Use consistent terminology

Choose one term for each important concept.

For example, do not alternate between:

* access token;
* OAuth token;
* bearer token;
* authentication token;

unless the distinction is meaningful.

Define an abbreviation at first use when it may not be familiar.

**Example:**

> Client ID Metadata Document (CIMD)

Then use `CIMD` consistently.

Preserve official capitalization for products, protocols, standards, and architectural patterns.

---

## 13. Avoid ambiguous pronouns

A pronoun should have one obvious referent.

**Avoid:**

> The adapter calls the service after it validates the token.

It is unclear whether "it" refers to the adapter or the service.

**Prefer:**

> The adapter validates the token before it calls the service.

---

## 14. Prefer concrete statements

Describe what happens rather than what something is intended to do.

**Prefer:**

> The client sends the `conversation_id` with each subsequent request.

**Avoid:**

> The client is designed to support conversation continuity.

The second statement may be useful as an additional explanation, but it should not replace the concrete behavior.

---

## 15. Use lists for multiple independent items

Do not put long sequences into a single sentence.

**Prefer:**

1. Validate the token.
2. Check domain authorization.
3. Execute the application workflow.
4. Return the result.

Use lists for:

* multiple consequences;
* requirements;
* alternatives;
* risks;
* steps;
* independent architectural constraints.

---

## 16. Use headings to separate decisions

Complex ADRs may contain several related architectural decisions.

Do not force all decisions into one paragraph.

**Prefer:**

```markdown
## Decision

### 1. MCP is a driving adapter

...

### 2. The application core remains independent of MCP

...

### 3. Token validation uses a dedicated port

...
```

Each subsection should contain one coherent architectural decision.

---

## 17. Keep paragraphs short

Prefer paragraphs of two to five sentences.

Use a new paragraph when the topic changes.

Do not split a tightly connected explanation into many fragments merely to satisfy a sentence limit.

---

## 18. State trade-offs explicitly

Architectural decisions normally have costs.

Use direct language:

> This approach removes duplicated business logic but prevents response streaming through the current MCP clients.

Avoid presenting a decision as universally superior.

**Avoid:**

> This approach is the best solution.

**Prefer:**

> This approach preserves the existing agentic workflow at the cost of response streaming.

---

## 19. Explain negative consequences

Every significant decision should be evaluated for downside.

Useful patterns include:

> The main limitation is...

> This introduces...

> The trade-off is...

> This requires...

> This prevents...

Example:

> The server is publicly reachable. This increases the exposed attack surface and requires strict token validation and rate limiting.

---

## 20. Avoid promotional language

An ADR is a decision record, not a product announcement.

Avoid:

* powerful;
* revolutionary;
* elegant;
* seamless;
* best-in-class;
* highly scalable;
* future-proof;
* cutting-edge;
* simple and robust;

unless the term has a precise technical meaning and is supported by evidence.

---

## 21. Avoid unnecessary certainty

Use precise language when information is incomplete.

Use:

* must;
* should;
* can;
* may;
* currently;
* expected;
* proposed;
* requires confirmation;

according to the actual certainty.

**Prefer:**

> The email claim must be present in the access token for the current authorization flow to work.

> The exact scope is still to be confirmed with the AAA team.

**Avoid:**

> MyLogin will provide the email claim.

when this has not been confirmed.

---

## 22. Separate current facts from target architecture

Use explicit markers when architecture is transitional.

**Current:**

> The existing application uses Streamlit as its driving adapter.

**Target:**

> The MCP server will be added as a second driving adapter.

**Fallback:**

> Until CIMD is available, the clients use statically registered OAuth clients.

This prevents future readers from confusing an existing capability with a planned one.

---

## 23. Preserve technical precision

Do not simplify a sentence if simplification changes its meaning.

For example:

> The Resource Server validates the JWT signature against cached JWKS keys.

is preferable to a simpler but less precise statement such as:

> The server checks the token.

Precision takes priority over stylistic simplicity.

---

## 24. Use normative language consistently

Use:

* **must** for mandatory constraints;
* **should** for strong recommendations;
* **may** for permitted behavior;
* **can** for capability;
* **must not** for prohibited behavior.

Do not use "should" when the architecture requires "must".

**Example:**

> The server must validate the token expiration.

---

## 25. Keep code and identifiers unchanged

Do not rewrite:

* class names;
* function names;
* file paths;
* API fields;
* protocol names;
* configuration keys;
* URLs;
* database objects;
* code snippets.

For example:

> `AccessTokenValidatorPort` validates the bearer token.

Do not change the identifier to make it linguistically simpler.

---

## 26. Prefer explicit references

When referring to another ADR, component, or section, use its name.

**Prefer:**

> The authorization model defined in ADR 0008 remains unchanged.

**Avoid:**

> The existing authorization model remains unchanged.

when multiple models exist.

---

## 27. Final STE editing pass

After the ADR content is complete, perform a separate language pass.

Check:

* Can each sentence be understood on the first reading?
* Does each sentence have a clear subject?
* Is the active voice used where appropriate?
* Are long sentences split when necessary?
* Are vague pronouns removed?
* Are unnecessary words removed?
* Are technical terms used consistently?
* Are facts, decisions, assumptions, and open points clearly distinguished?
* Are trade-offs stated explicitly?
* Is uncertainty represented accurately?
* Has technical precision been preserved?

Do not rewrite technically precise content merely to make it shorter.

The goal is **clear technical English, not simplistic English**.
