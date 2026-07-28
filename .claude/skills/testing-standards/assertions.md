# Assertions

## Core rule: Assert the actual value, not just existence

```typescript
// ❌ WEAK — passes when anything truthy is returned
expect(result).toBeTruthy();
expect(fn).toBeDefined();
expect(items.length).toBeGreaterThan(0);

// ✅ STRONG — verifies the actual thing
expect(result).toBe('expected-value');
expect(fn).toHaveBeenCalledWith('expected-arg');
expect(items).toHaveLength(3);
```

## Fake pass anti-patterns

### 1. `expect(true).toBe(true)` — always passes, tests nothing

```typescript
// ❌ Fake pass
if (button) {
  await user.click(button);
  expect(mock).toHaveBeenCalled();
} else {
  expect(true).toBe(true); // passes even when button never renders
}

// ✅ Fail fast if element absent
expect(button).toBeInTheDocument();
await user.click(button);
expect(mock).toHaveBeenCalledWith('expected-arg');
```

### 2. Silent catch with no assertion

```typescript
// ❌ Error path untested — catch swallows everything
try {
  await result.current.deleteFile(fileId);
} catch { /* expected */ }

// ✅ Assert post-error state
try {
  await result.current.deleteFile(fileId);
} catch {
  expect(result.current.files.length).toBe(1); // not removed on error
}
expect(mockDeleteApi).toHaveBeenCalled();
```

### 3. Conditional assertion that passes vacuously

```typescript
// ❌ If element doesn't render, test silently passes
if (downloadButton) {
  await user.click(downloadButton);
  expect(mockOpen).toHaveBeenCalled();
}

// ✅ Assert existence first
const downloadButton = screen.getByRole('button', { name: /download/i });
await user.click(downloadButton);
expect(mockOpen).toHaveBeenCalledWith('https://cdn.example.com/file.pdf', '_blank');
```

## Exact counts over ranges

```typescript
// ❌ Accepts 1, 100, anything above 0
expect(screen.getAllByRole('listitem').length).toBeGreaterThan(0);

// ✅ Verifies exact expected count
expect(screen.getAllByRole('listitem')).toHaveLength(3);
```

Only use `toBeGreaterThan` / `toBeGreaterThanOrEqual` when the count is genuinely variable — and document why.

## Navigation assertions

```typescript
// ❌ Too vague
expect(mockRouter.push).toHaveBeenCalled();

// ✅ Exact destination
expect(mockRouter.push).toHaveBeenCalledWith('/expected/path');
```

## Text assertions

```typescript
// ❌ Brittle — breaks on translation changes
expect(screen.getByText('Transfer berhasil')).toBeInTheDocument();

// ✅ Use testIds for stability across i18n
expect(screen.getByTestId('success-banner')).toBeInTheDocument();
```
