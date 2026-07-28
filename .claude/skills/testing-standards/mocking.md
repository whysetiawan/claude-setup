# Mocking

## Rule: Mock at system boundaries only

Mock HTTP calls, external services, and platform APIs. Let your own code run real.

```typescript
// ✅ RIGHT — real hook runs, HTTP is intercepted
server.use(
  http.get('/api/distributions', () =>
    HttpResponse.json([{ status: 'succeeded', amount: 5000 }])
  )
);
render(<FeatureComponent />);
await waitFor(() => expect(screen.getByTestId('result-card')).toBeInTheDocument());
```

```typescript
// ❌ WRONG — mocking your own hook hides its entire logic
jest.mock('@/hooks/useFeatureData', () => ({
  useFeatureData: jest.fn(() => ({ data: mockData, isLoading: false })),
}));
```

The mocked hook hides: query state, error handling, loading states, data transformation bugs, caching.

## What you CAN mock

| Target | Reason |
|---|---|
| HTTP endpoints (via MSW/nock/fetch-mock) | System boundary — real behavior |
| External SDK calls (analytics, payments, auth) | Third-party, not your behavior |
| Browser APIs absent in test env (`matchMedia`, `IntersectionObserver`, `clipboard`) | Not available in jsdom |
| `Date.now()` / `Math.random()` | Determinism |
| `localStorage` / `sessionStorage` | Control test state |

## What you CANNOT mock

- Your own hooks
- Your own service/repository functions
- Your own domain model methods
- Anything you wrote and control

## Designing for testability

Pass dependencies in rather than constructing them inside functions:

```typescript
// Easy to test
function processOrder(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to test — creates its own dependency
function processOrder(order) {
  const client = new StripeClient(process.env.KEY);
  return client.charge(order.total);
}
```

Prefer specific API functions over generic fetchers — each function is independently mockable at the HTTP level.

## Error path testing

Always test error states by making the HTTP mock return 4xx/5xx:

```typescript
server.use(
  http.post('/api/orders', () =>
    HttpResponse.json({ error: 'Validation failed' }, { status: 400 })
  )
);
// Then assert the component handles it gracefully
```
