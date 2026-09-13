# Deprecation

Three releases, and the middle one is what makes the removal safe. For the rule, see the principle **Deprecate in three steps, never in one** in `SKILL.md`.

## Step 1: works, warns

The prop keeps working exactly as before. Add the JSDoc tag so editors strike it through, and warn once in development.

```tsx
type ButtonProps = {
  /** @deprecated Use `appearance` instead. Removed in v3. */
  variant?: "primary" | "secondary";
  appearance?: "solid" | "outline" | "ghost";
};

if (process.env.NODE_ENV !== "production" && variant !== undefined) {
  warnOnce(
    "Button: `variant` is deprecated, use `appearance`. " +
    'variant="primary" becomes appearance="solid". Removed in v3.'
  );
}
```

Three things the message must carry: the replacement, the mapping for at least one value and the release it disappears in. A message saying only "`variant` is deprecated" sends the consumer to a changelog they will not read.

`warnOnce` is keyed on the message, not called per render. A warning firing on every render of a list gets muted, and muted warnings do not migrate anyone.

Where the old and new prop are both passed, the new one wins and the warning says so. Silently preferring the old one strands consumers mid-migration.

## Step 2: works, fails the type check

A minor release later, the runtime behavior is unchanged and the type is gone.

```tsx
type ButtonProps = {
  appearance?: "solid" | "outline" | "ghost";
  /** @deprecated Removed from the type. Runtime support ends in v3. */
} & { variant?: never };
```

`variant?: never` makes any passed value a type error while the implementation still handles it. A TypeScript consumer sees a red squiggle at the call site instead of a console message, and a JavaScript consumer keeps running.

This step is what makes the removal in step 3 boring. Skipping it means step 3 is the first time anyone notices.

## Step 3: removed

A major release. Delete the prop, the handling and the warning. The changelog entry carries the mapping table, because someone is upgrading two majors at once.

| Removed | Replacement |
| --- | --- |
| `variant="primary"` | `appearance="solid"` |
| `variant="secondary"` | `appearance="outline"` |

## Ship a codemod for anything mechanical

A rename is mechanical, so migrate it rather than asking consumers to. `jscodeshift` handles the common cases in under thirty lines:

```js
module.exports = function (file, api) {
  const j = api.jscodeshift;
  const map = { primary: "solid", secondary: "outline" };

  return j(file.source)
    .findJSXElements("Button")
    .find(j.JSXAttribute, { name: { name: "variant" } })
    .forEach((path) => {
      const value = path.node.value.value;
      if (!map[value]) return;                       // leave dynamic values alone
      path.node.name = j.jsxIdentifier("appearance");
      path.node.value = j.stringLiteral(map[value]);
    })
    .toSource();
};
```

Two things it must not do. It never touches a value it does not recognise, including a dynamic `variant={x}`, and it never renames a `Button` imported from somewhere else. Both cases go in the report for a human, and the codemod's output is a diff to review rather than a commit.

## Deprecating a whole component

Same three steps, with the shim in the middle:

1. The old component re-exports the new one, adapting props, and warns.
2. The old export is typed `never` or moved to a `/deprecated` entry point consumers must opt into.
3. Removed in a major.

Where the replacement is not a drop-in, do not adapt. Warn with the migration note and leave the old implementation untouched, because a shim that changes behavior is worse than one that does not exist.

## The rule that makes all of this cheap

Deprecation cost is proportional to how public the prop is. An internal prop on a component nobody outside the repo imports can be renamed in one commit with a grep.

So check the exported entry point before starting a deprecation dance. Half the props in a design system are not API, and treating them as API is its own kind of expensive.
