---
name: react-useeffect-guide
description: >
  Effect discipline for React code. Use when writing, reviewing, or refactoring
  useEffect, cleanup functions, dependency arrays, useLayoutEffect, data fetching
  in Effects, or synchronization with external systems.
---

# React useEffect Guide

## Golden rule

`useEffect` is for **synchronizing a component with an external system**. If no
external system is involved, remove the Effect.

External systems include browser APIs, network connections, timers,
third-party widgets, external stores, and raw data fetching when a framework or
cache library is not available.

## Workflow

### 1. Classify the Effect

For each Effect, name the external system it synchronizes with.

If there is no external system, replace the Effect with the React-native shape:

- Derived value → calculate during render, or `useMemo` only when measurement
  shows the calculation is expensive.
- User-triggered work → event handler.
- Reset on identity change → split the component and pass `key`.
- Parent/sibling synchronization → lift state or update both states in the same
  event.
- External store subscription → `useSyncExternalStore`.
- Repeated Effect pattern → custom hook.

**Complete when:** every Effect has a named external system, or has been replaced
with a non-Effect shape.

### 2. Make setup and cleanup symmetrical

A valid Effect has one setup story and, when setup creates ongoing work, a cleanup
that mirrors it:

```js
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [serverUrl, roomId]);
```

Cleanup-only Effects are suspicious. In Strict Mode, setup → cleanup → setup runs
once extra in development; the user should not be able to tell.

**Complete when:** every subscription, timer, listener, observer, connection,
request guard, or widget setup has a matching teardown or cancellation guard.

### 3. Prove the dependencies

Dependencies are not chosen; they are determined by the reactive values read by
the Effect: props, state, and variables/functions declared inside the component.

Rules:

- List every reactive value used by the Effect.
- Do not suppress `react-hooks/exhaustive-deps`; change the code instead.
- To remove a dependency, remove the reactive read: move object/function creation
  inside the Effect, use a functional state updater, or separate non-reactive
  logic with the project-approved Effect Event pattern.

**Complete when:** the dependency array matches the Effect body and the hooks
linter would not need suppression.

### 4. Handle data fetching deliberately

Prefer framework loading, route loaders, or cache libraries such as TanStack
Query/SWR. Use raw Effect fetching only when those are not in scope.

Raw Effect fetching needs a stale-response guard:

```js
useEffect(() => {
  let ignore = false;

  async function fetchData() {
    const result = await fetchBio(person);
    if (!ignore) setBio(result);
  }

  fetchData();
  return () => { ignore = true; };
}, [person]);
```

**Complete when:** fetch responses cannot update state after being superseded,
and any missing caching, SSR, deduplication, or waterfall concern has been called
out.

### 5. Escalate only when needed

Use `useLayoutEffect` only for visual DOM reads/writes where a paint before the
Effect would cause visible flicker. Otherwise use `useEffect`.

Extract a custom hook when the same synchronization pattern appears in more than
one component, or when hiding imperative setup makes the component declarative.

**Complete when:** the hook choice is justified by observable behavior, not habit.

## Reference pointer

For examples of anti-patterns, dependency repairs, cleanup pairs, custom hooks,
and server-rendering caveats, read
[references/useeffect-patterns.md](references/useeffect-patterns.md) only when a
case needs that detail.

## Sources

- [React Docs: useEffect Reference](https://react.dev/reference/react/useEffect)
- [React Docs: You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
