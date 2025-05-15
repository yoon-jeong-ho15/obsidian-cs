# 개념
https://react.dev/warnings/invalid-hook-call-warning

- ✅ Call them at the top level in the body of a [function component](https://react.dev/learn/your-first-component).
- ✅ Call them at the top level in the body of a [custom Hook](https://react.dev/learn/reusing-logic-with-custom-hooks).

It’s **not** supported to call Hooks (functions starting with `use`) in any other cases, for example:

- 🔴 Do not call Hooks inside conditions or loops.
- 🔴 Do not call Hooks after a conditional `return` statement.
	- [[A change in the order of Hooks]]
- 🔴 Do not call Hooks in event handlers.
- 🔴 Do not call Hooks in class components.
- 🔴 Do not call Hooks inside functions passed to `useMemo`, `useReducer`, or `useEffect`.

