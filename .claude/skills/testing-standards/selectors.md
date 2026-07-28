# Selectors

## Priority order

Query elements in this order — stop at the first that works:

1. **`data-testid` / `data-qaid`** — stable under content changes and i18n
2. **`getByRole`** — semantic, accessible, survives DOM restructuring
3. **`getByLabelText`** — for form inputs
4. **`getByText`** — last resort; breaks on translation changes

## Anti-patterns

### CSS class selectors

```typescript
// ❌ Breaks when designer renames a Tailwind class
container.querySelector('.bg-[#00372F]');
expect(element).toHaveClass('text-jade-green');

// ✅ Use testId or role
screen.getByTestId('otg-status-banner');
screen.getByRole('status', { name: /otg/i });
```

### Tag + class selectors

```typescript
// ❌ Breaks on HTML restructuring
container.querySelectorAll('b[class*="text-xs"]');
container.querySelector('span.line-through');

// ✅ Query by content or testId
screen.getByText('nomor rekening');
screen.getByTestId('original-fee');
```

### Generic tag selectors

```typescript
// ❌ Matches any button on the page
container.querySelector('button');

// ✅ Role + accessible name
screen.getByRole('button', { name: /confirm/i });
```

## Exception: file inputs

`input[type="file"]` has no accessible role. `container.querySelector('input[type="file"]')` is acceptable — but always add an exact count assertion:

```typescript
const fileInputs = container.querySelectorAll('input[type="file"]');
expect(fileInputs).toHaveLength(4); // desktop + camera + gallery + file picker
```

## Adding testIds to source

When no stable selector exists, add a `data-testid` to the component:

```tsx
<div data-testid="success-banner" className="...">
```

This is preferable to coupling tests to CSS implementation details. testIds are a contract between the component and its tests.

## Interaction before asserting existence

```typescript
// ❌ Crashes with "null is not an element" if button doesn't render
await user.click(container.querySelector('button'));

// ✅ Assert existence first — gives useful error if missing
const btn = screen.getByRole('button', { name: /submit/i });
expect(btn).toBeInTheDocument();
await user.click(btn);
```

## Rule: try userEvent.press first, fall back to fireEvent.press for gesture-handler Pressables

Default to `await user.press(el)` (the `userEvent` instance from `render()`) — it's closer to real user interaction. Only fall back to `fireEvent.press(el)` if `userEvent.press()` silently no-ops.

The failure mode is specific to custom `Pressable` wrappers built on `react-native-gesture-handler` (common in design-system libraries) — they don't emit whatever gesture sequence `userEvent` expects, so `onPress` never fires. Nothing errors: the element is present, the assertion after it just fails or the mock stays uncalled, with no indication `userEvent` itself was the problem. Plain RN `Pressable`/`TouchableHighlight`-based wrappers (no gesture-handler underneath) work fine with `userEvent.press()` — verify by checking what the component renders under the hood before assuming it needs the fallback.

```typescript
// ✅ Try first — works for plain RN Pressable/TouchableOpacity/TouchableHighlight wrappers
await user.press(button);

// ✅ Fallback — only needed for react-native-gesture-handler-based Pressables,
// where userEvent.press() silently no-ops
fireEvent.press(button);
```
