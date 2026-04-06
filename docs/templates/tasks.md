# {Domain} Task List

## Legend
- **Priority**: P0 (blocker) / P1 (must) / P2 (should) / P3 (nice-to-have)
- **Type**: backend / frontend / migration / infra / test
- **Status**: checkbox

## Tasks

### Phase 1: Backend / API
- [ ] A-{domain}-001: (task) — Type: backend, Priority: P1, Depends: none
- [ ] A-{domain}-002: (task) — Type: backend, Priority: P1, Depends: A-{domain}-001

### Phase 2: Frontend / UI
- [ ] B-{domain}-001: (task) — Type: frontend, Priority: P1, Depends: A-{domain}-001
- [ ] B-{domain}-002: (task) — Type: frontend, Priority: P1, Depends: B-{domain}-001

### Phase 3: Integration / Polish
- [ ] C-{domain}-001: (task) — Type: test, Priority: P1, Depends: B-{domain}-001

<!-- Add/remove phases as needed. Not all domains follow this exact structure. -->
