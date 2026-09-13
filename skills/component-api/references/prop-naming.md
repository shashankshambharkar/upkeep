# Prop naming

Prop names are the library's vocabulary, and a vocabulary only works when it is one vocabulary. For why names outlive implementations, see the opener in `SKILL.md`.

## One word per concept

| Concept | Prop | Not |
| --- | --- | --- |
| Visual treatment | `appearance` | `variant`, `kind`, `type`, `style` |
| Semantic colour | `tone` | `color`, `intent`, `status`, `severity` |
| Physical scale | `size` | `scale`, `dimension`, `sizing` |
| Information density | `density` | `compact`, `spacious`, `condensed` |
| Visual weight | `emphasis` | `bold`, `strong`, `prominence` |
| Fills its container | `fullWidth` | `block`, `stretch`, `wide` |
| Non-interactive | `disabled` | `inactive`, `readonly` where it is not truly read-only |
| In-flight | `loading` | `pending`, `busy`, `isLoading` |
| Cannot be edited but is focusable | `readOnly` | `locked`, `frozen` |

The right column is not wrong in isolation. `variant` is a fine name, and a library that has settled on `variant` should grep `appearance` to zero instead. What breaks a library is holding both.

`type` deserves a specific refusal. It collides with the native `type` attribute on `button` and `input`, so a component accepting both has to disambiguate at every call site.

## Booleans read true when present

Name a boolean so that its absence is the default and its presence reads correctly in JSX:

```tsx
<Button disabled />        // correct: absent means enabled
<Dialog open />            // correct: absent means closed
<Input hidden={false} />   // wrong: the default needs an explicit false
```

A boolean whose default is `true` forces every ordinary call site to pass `false`. Invert the name. `hidden` defaulting true becomes `visible`, or better, the parent stops rendering it.

Drop the `is` and `has` prefixes. React's own props are `disabled` and `checked`, and a library mixing `disabled` with `isLoading` has two conventions in one signature.

## Handlers name the intent or the event, never both

`onChange`, `onSubmit`, `onDismiss` name what happened in the component's own terms. `onClick`, `onKeyDown` name a DOM event and carry its native payload.

Pick one per interaction. `onSubmitClick` is both, and it tells a consumer neither what fired nor what they receive.

Pass the meaningful value first and the event second, because the value is what consumers use:

```ts
onChange: (value: string, event: React.ChangeEvent<HTMLInputElement>) => void
```

A handler that passes only the event makes every consumer write `e.target.value`, which is the component's job.

## Slots take nodes and are named for the position

```tsx
<Card
  media={<img src={src} alt="" />}
  actions={<Button>Save</Button>}
>
  Body copy lives in children.
</Card>
```

`media` and `actions` name where the node lands. `renderMedia` names a function, which is composition wearing a prop's clothes, and belongs in children instead.

A slot takes `ReactNode`. The moment it needs an argument it is a render prop, and the component is a layout whose consumers want the pieces.

## Native props pass through, and never get renamed

A component wrapping an `input` accepts `placeholder`, `autoFocus`, `name` and `required` under those exact names. Renaming a native prop means every consumer learns your synonym for something they already know.

Where a component must intercept a native prop, keep the name and document the interception. `onChange` on a controlled input is intercepted by nearly every library, and calling it `onValueChange` is a worse fix than documenting it.

## Reserve `data-` for state the CSS needs

```tsx
<button data-appearance={appearance} data-tone={tone} data-loading={loading || undefined}>
```

Attributes rather than classes give CSS a selector per axis without a class-name join, and they show the component's current cell in devtools. Use `|| undefined` on booleans so `data-loading="false"` never renders, since CSS treats the presence of the attribute as true.
