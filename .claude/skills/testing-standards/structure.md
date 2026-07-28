# Test Structure

## Two-level nesting

Top describe = **user journey step**. Nested describe = **technical domain or variant**.

```
describe('CheckoutPage')
  describe('on mount: cart summary loads')
    describe('happy path')
    describe('empty cart')
    describe('network error')

  describe('form interactions')
    describe('shipping address')
    describe('payment method selection')

  describe('order submission')
    describe('happy path')
    describe('payment declined')
    describe('validation errors')

  describe('promo code')
    describe('apply flow')
    describe('invalid code')
```

## Rules

**Every `it()` belongs inside a semantic `describe`.** No orphan top-level tests.

**Names describe behavior, not code.**

```typescript
// ❌ Implementation-oriented
it('covers line 142 — GPS path error handling', ...);
it('handleSubmit when useGpsPath=true branch', ...);
describe('lines 60-85: fallback quotation creation', ...);

// ✅ Behavior-oriented
it('shows validation error when required field is empty', ...);
it('redirects to payment on successful submission', ...);
describe('when GPS submission fails', ...);
```

**No line numbers, branch names, or coverage references in test names.** Ever.

## AAA pattern inside each test

```typescript
it('disables submit when required field is empty', async () => {
  // Arrange
  render(<Form />);
  await waitFor(() => expect(screen.getByTestId('submit')).toBeInTheDocument());

  // Act — no action (testing initial state)

  // Assert
  expect(screen.getByTestId('submit')).toBeDisabled();
});
```

## Test isolation

Each `it()` must be fully independent — runnable in any order, in any subset.

```typescript
beforeEach(() => {
  jest.clearAllMocks();
  // Reset any shared state: cache, sessionStorage, mocked return values
});

afterEach(() => {
  // Clean up side effects if needed
});
```

No shared mutable variables that one test sets and another test reads.

## What goes in which level

| Level | Purpose | Example |
|---|---|---|
| Top `describe` | The component/page under test | `describe('LoginPage')` |
| Second `describe` | Journey step or feature area | `describe('OTP verification')` |
| Third `describe` | Variant/path | `describe('when OTP expired')` |
| `it` | One observable behavior | `it('shows error message')` |

## Anti-patterns

```typescript
// ❌ Coverage-named describe blocks
describe('100% coverage edge cases', ...);
describe('branch: isGpsPath=false', ...);

// ❌ Orphan top-level it()
describe('LoginPage', () => {
  it('renders correctly', ...); // ← no parent describe for journey step
  describe('form interactions', () => { ... });
});
```
