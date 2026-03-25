# Working agreement for this repo

Before implementing any code change:

1. Read `/docs/manifesto.md`
2. Read `/ai/architecture-constraints.md`
3. Read the relevant `/spec/entities/*.yaml`
4. Read the relevant `/spec/workflows/*.yaml`
5. Summarize the applicable constraints and invariants
6. State assumptions explicitly
7. Only then propose or write code

Mandatory checks:

- No tenant-unscoped data access
- No direct workflow-state mutation outside workflow services
- No buyer mutation of vendor submissions
- No approval bypass
- No deadline enforcement in client-only code
- No technical/commercial visibility leaks
- No hard delete of core transactional records
- No undocumented business behavior changes without spec updates

Before finalizing:

- Explain which constraints were applied
- List assumptions
- Update relevant specs if behavior changed
- Run relevant tests
- Confirm affected invariants are still enforced