# Production Hardening

A design that only works with perfect data isn't production-ready. Harden against the inputs, errors, languages, and network conditions real users actually throw at it.

## Test with extreme inputs before calling it done

Very long text, very short/empty text, special characters (emoji, RTL, accents), large numbers, 1000+ list items, zero data. If any of these weren't part of the test pass, the feature isn't hardened yet.

## Text overflow & wrapping

```css
/* Single line, ellipsis */
.truncate { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

/* Multi-line clamp */
.line-clamp { display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; overflow: hidden; }

/* Allow wrapping instead of overflowing */
.wrap { word-wrap: break-word; overflow-wrap: break-word; hyphens: auto; }

/* Flex/Grid children overflow their container by default — fix with: */
.flex-item, .grid-item { min-width: 0; }
```

## Internationalization, technically

**Logical properties instead of left/right** — makes RTL automatic instead of a separate pass:

```css
margin-inline-start: 1rem;   /* not margin-left */
padding-inline: 1rem;        /* not padding-left/right */
border-inline-end: 1px solid;/* not border-right */
```

**Text expansion budget**: German +30%, French +20%, Finnish +30-40%, Chinese -30% (fewer characters, same visual width). Never size a button to fit English text exactly:

```jsx
// fragile: assumes short English text
<button className="w-24">Submit</button>
// resilient: sized by content
<button className="px-4 py-2">Submit</button>
```

**Use Intl APIs, not hand-rolled formatting** — date/number/currency formats and pluralization rules vary by locale in ways string interpolation can't capture:

```js
new Intl.DateTimeFormat('de-DE').format(date);
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(1234.56);
```

## Error handling by status code

| Status | Show |
|---|---|
| 400 | Inline validation errors, preserve user input |
| 401 | Redirect to login |
| 403 | Permission error — explain why, what to do instead |
| 404 | Clear "not found" state, not a blank page |
| 429 | Rate-limit message with retry guidance |
| 500 | Generic apology + retry button + support path — never expose the stack trace |

Network failures need a retry button and an explanation, not a dead end. Form errors must preserve what the user typed — never wipe the form on a failed submit.

## Edge cases checklist

- **Empty states**: no items / no search results / no permission / error — each needs a distinct message and a next action, not the same blank "No items" for all of them
- **Large datasets**: pagination or virtualization — never render 10,000 items unvirtualized
- **Concurrent operations**: disable the button while a request is in flight to prevent double-submit; handle race conditions explicitly
- **Permission states**: read-only mode and "no access" need their own UI, not a silently broken interactive one

## Debounce/throttle high-frequency events

```js
const debouncedSearch = debounce(handleSearch, 300); // input
const throttledScroll = throttle(handleScroll, 100); // scroll
```

Clean up: cancel pending requests/timers/subscriptions on unmount — uncancelled work is the most common source of memory leaks and "setState on unmounted component" warnings.

## Graceful degradation

Core functionality should survive a missing/late stylesheet or script where feasible. Images need real alt text regardless of whether the image loads. A single failed component should show its own error boundary, not take down the whole page.

**Never**: trust client-side validation alone (always re-validate server-side), assume English-length text fits any fixed width, ship a generic "Error occurred" when the status code tells you exactly what happened, ignore the offline/slow-3G case.
