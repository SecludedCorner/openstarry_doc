<!-- Status: CURRENT -->
<!-- Layer: 2-Philosophy -->
<!-- Applies to: v0.44.0-alpha (Plan44+) -->

# 69. Rule #63: L4 — Config Activates

`[Cycle 03-8 新增: D8-Q2, D8-Q37]`

## Background

Rule #58 established hypothesis inversion for new-feature Plan verification: H0 = fabrication present. The three-layer innocence framework (L1/L2/L3) was designed to catch the first three generations of fabrication:

| Layer | Catches | Method |
|-------|---------|--------|
| L1: Code Exists | Gen1: Phantom production | File exists + diff matches spec |
| L2: Code Works | Gen2: Phantom test | Tests exist + cover feature |
| L3: Code Integrates | Gen3: Phantom integration | Import chain + call site + used |

## The Gen4 Gap

In Cycle 03-8, GUARDIAN (#11) discovered that COND-4's factory-level test (test:471) verified the arbiter's `id` but NOT that `coldStartGear=2` produces `evaluate() → gear 2`. This matches TURING's Gen4 prediction: **configuration-layer fabrication** — code is correct, config-to-behavior path is not verified.

L1+L2+L3 alone cannot catch this pattern because:
- L1: The config type exists (PASS)
- L2: A test exists that creates the plugin with the config (PASS)
- L3: The factory imports and uses the config (PASS)
- **But**: No test verifies that the config value actually changes runtime behavior

## Rule #63 Definition

**L4: Config Activates** — For any new configurable parameter, at least one test must verify the full `config → factory → runtime behavior` chain produces the expected behavioral outcome.

### Criteria

1. The test MUST start from the public plugin creation function (e.g., `createGearArbiterDynamicPlugin()`)
2. The test MUST pass a non-default configuration value
3. The test MUST verify an observable behavioral outcome (not just structural properties like `id` or `type`)
4. The test MUST NOT construct internal classes directly (this bypasses the config→factory chain)

### Application

Rule #63 applies to:
- All new configurable parameters in any plugin
- All config fields that affect runtime behavior
- Both existing parameters (when discovered untested) and new parameters

Rule #63 does NOT apply to:
- Type-only definitions (e.g., Phase3Config placeholder with no runtime effect)
- Documentation-only changes
- Parameters that do not affect observable behavior

## Relationship to Other Rules

- **Rule #58**: H0 = fabrication present. L4 strengthens the evidence required to reject H0.
- **Rule #60**: ENG-FAB checklist. H section (Configuration) verifies config existence; L4 verifies config behavior.
- **Rule #54**: A-9 integration verification. L4 is complementary but distinct — A-9 checks wiring, L4 checks behavioral outcome.

## MR-11 Alignment

Configuration completeness is part of complete architectural practice. A configurable parameter that doesn't change behavior is a false promise — it violates the spirit of Tenet #3 (五蘊 completeness) by creating the appearance of configurability without the substance.

---

*Document added in Cycle 03-8*
*Source: R3 decisions D8-Q2, D8-Q37*
