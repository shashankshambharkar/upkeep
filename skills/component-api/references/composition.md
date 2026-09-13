# Composition

The three patterns that replace a prop once a prop stops being enough, in the order to reach for them. For the threshold, see the principle **Configuration until the tree shape changes, composition after** in `SKILL.md`.

## Slots, for a known position

The cheapest pattern, and the right one when the component owns the layout and the consumer owns the content.

```tsx
type CardProps = {
  media?: React.ReactNode;
  actions?: React.ReactNode;
  children: React.ReactNode;
};
```

The component decides where `media` lands, what it is wrapped in and how it responds at each breakpoint. The consumer decides what it is. Neither has to know the other's decision.

Reach for a slot the moment a prop starts describing a node. `mediaUrl`, `mediaAlt`, `mediaAspect` and `mediaLoading` are four props reconstructing an `img` badly, and one `media` slot does it correctly.

## Compound components, for a layout the consumer arranges

When the consumer needs to control order, omit parts, or put their own nodes between them, expose the parts.

```tsx
<Dialog>
  <Dialog.Trigger>Delete</Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Title>Delete this project?</Dialog.Title>
    <Dialog.Description>This cannot be undone.</Dialog.Description>
    <Dialog.Footer>
      <Dialog.Close>Cancel</Dialog.Close>
      <Button tone="danger">Delete</Button>
    </Dialog.Footer>
  </Dialog.Content>
</Dialog>
```

Shared state travels through context, so the parts coordinate without the consumer wiring anything. That is the whole reason the pattern exists, and a compound component whose parts do not share state is a namespace.

Two costs to accept before choosing it. The consumer can arrange the parts wrongly, so guard the arrangements that break accessibility by throwing from the context hook when a part renders outside its provider. And every exposed part is API forever, so expose the parts the product arranges and not every internal node.

## `asChild`, for changing the element without changing the component

A button that needs to be a link keeps its styling, its variants and its focus behavior, and changes its element:

```tsx
<Button asChild>
  <Link href="/settings">Settings</Link>
</Button>
```

The component merges its props, class names and ref onto the single child instead of rendering its own element. Radix ships this as `Slot`, and a hand-rolled version is about fifteen lines of `cloneElement`.

It replaces the `as` prop, which takes a component type and drags generic type parameters through the whole signature. `asChild` keeps the types simple, because the child is just a node.

Two rules. It takes exactly one child, so throw on zero or many. And it is not a licence for a semantic change: `asChild` onto a `div` gives you a styled div with none of a button's keyboard behavior, and the fix is a real button.

## Which one

| Situation | Pattern |
| --- | --- |
| Consumer supplies content, component owns placement | Slot |
| Consumer arranges parts and may omit some | Compound |
| Consumer keeps the component, changes the element | `asChild` |
| Consumer needs a value from internal state to render | Render prop, and reconsider the component |

A render prop is the pattern of last resort, correct only when the consumer genuinely cannot render without state the component holds, as in a virtualised list passing an index. Three render props on one component means the component is a layout, and its consumers want the pieces.

## Do not publish internal structure

```tsx
// Wrong: every internal node is now API, and a refactor breaks consumers.
<Card classNames={{ root: "...", header: "...", media: "...", footer: "..." }} />
```

The consumer is now coupled to a DOM tree you wanted to be free to change. Ship `className` on the root. Add a single named part prop only where a consumer demonstrably cannot reach a part any other way, one at a time, each with a reason in the changelog.
