# Demo script — Notary workflow agent (DACH)

**Length:** 10 min + 5 min Q&A (longer than other demos — this is the governance flagship)
**Presenters:** Engineer lead + Daniel (technical) + Titus (commercial). Martin on standby for regulatory questions.

## Flow

### 1. Scope recap (45s)
Frame the supervision promise: agent never signs, never posts, never judges. It prepares, flags, and logs.

### 2. Clean intake (90s)
Drop a clean real estate purchase intake. Show:
- Matter type classified
- ID validated
- Sanctions cleared
- Template populated
- Trust entry staged
- Supervision log complete and hash-chained
- Notary sees a prepared packet ready for review

### 3. The missing field (60s)
Show a corporate formation intake missing the shareholder breakdown. Agent flags with specific pointer. No attempt to guess. Staff member sees exactly what's needed.

### 4. The sanctions hit (90s)
Show a matter where the sanctions screen returns a hit. Agent halts. Audit log captures halt with reason. Partner receives notice. This is the hard-stop proof.

### 5. The ID mismatch (60s)
Show an ID document that doesn't match submitted client name (Bäcker vs. Becker — umlaut difference). Agent catches two independent signals: (1) ID validation confidence 0.94, below the 0.95 threshold. (2) Name mismatch between ID and intake. Either signal alone triggers halt. Show both in the audit log — defense in depth.

### 6. The audit trail (120s — DO NOT RUSH)
Walk through the hash-chained supervision log for the clean matter. Show:
- Every action recorded
- Model version + prompt hash per action
- Previous-record hash links
- Signature on each record
- Exportable as a regulatory-review packet

State plainly: "If Dienstaufsicht asks why the agent did X on matter Y, here it is. One file. Signed. Immutable."

### 7. Kill switch (45s)
Flip the kill flag mid-intake. Show halt. Show audit log records halt event with timestamp and partner who triggered it.

### 8. What the prospect buys (45s)
Titus: $25K Standard tier, 60 days, one workflow (matter intake), audit-ready from day one. DACH notary pedigree delivered by the people who built Bundesnotarkammer infrastructure.

## Q&A prep

- **"Is this AI Act compliant?"** EU AI Act classifies this as high-risk per Annex III (administration of justice, legal advice adjacent). The agent is designed for that path: human oversight, logging, transparency to supervisor. Production cutover includes a DPIA and conformity documentation.
- **"What about DSGVO Art. 22?"** Agent never makes a decision with legal effect. Notary decides, agent prepares. Every decision point routes to a human.
- **"GoBD and retention?"** Audit log is append-only, signed, 10-year retention by default. Exportable to any archival system.
- **"What if the model hallucinates a matter classification?"** Confidence threshold 0.90 with hard fallback to human. Every classification is logged with its reasoning for review.
- **"Can the audit log be tampered with?"** Ed25519-signed, hash-chained. Any tamper breaks the chain and is detectable on verification.
- **"Who holds the signing key?"** Firm holds it. Civic does not. Part of the 60-day setup.
- **"What about beA / notary eNK integration?"** Out of MVP scope. Phase 2 conversation.

## Don't
- Don't claim the agent "complies with the AI Act." It's designed to support compliance. The firm is the responsible operator.
- Don't skip the hash-chain walk. That's the single most important proof for a regulated buyer.
- Don't demo the model doing legal reasoning. That's the one thing it must never do.
- Don't use real client data under any circumstance, even synthetic. Keep samples obviously fake.
