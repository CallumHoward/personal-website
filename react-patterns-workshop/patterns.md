# Add `className` to allow Styled Components styling

--

- **Pattern Type:** `Styled Components`
- **References:** [Styled Components docs](https://styled-components.com/docs/basics#styling-any-component)
- **Code example:** [NotificationBadge](https://github.com/SafetyCulture/frontend-reactor/blob/6c7c28b64ad7c3c9f723126afa925a9f6f610249/sc-web-ui/packages/react/badge/notification-badge/notification-badge.tsx#L17)

### Description

When styling a component using the `styled(Component)` syntax, inside the definition of `Component` it is necessary to pass the `className` prop to an underlying native HTML element.

### ✍️ Exercise: Enable styling of a Link component

<details>
  <summary>Click to expand exercise</summary>

1. Enable styling on the second link
2. Links should have underline style without any CSS changes  
   (hint: underline is applied on `a` if `href` attribute is defined)

</details>

--

<iframe
  src="https://stackblitz.com/edit/vitejs-vite-zdwien?ctl=1&embed=1&file=src%2FApp.tsx&hideExplorer=1&hideNavigation=1"
  style="width:100%; height: 600px; border:0; border-radius: 4px; overflow:hidden;"
  title="Workshop: Styled Components className example"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

---

# Get literal union string values as an array

- **Pattern Type:** `Typescript`
- **Code example:** [Badge](https://github.com/SafetyCulture/frontend-reactor/blob/6c7c28b64ad7c3c9f723126afa925a9f6f610249/sc-web-ui/packages/react/badge/types.ts#L19-L27)

### Description

The situation is that you want to utilise a string literal union type, and then iterate through all values of the type. A naive approach would be to redefine the values of the union in an array:

### Example: Naive approach ❌

- 🙁 Duplication of union values
- 🙁 Prone to error if union is updated

```tsx [] []
// types.ts
export type Variant = "accent" | "negative" | "warning" | "info";

// component.stories.tsx
const variants: Variant[] = ["accent", "negative", "warning", "info"];
export const ComponentVariants: Story = () => (
  <>
    {variants.map((variant: Variant) => (
      <Component key={variant} variant={variant} />
    ))}
  </>
);
```

<aside class="notes">
There is some duplication here that we can improve upon. Consider also that the types might be defined in a separate file. If a new variant gets added to the type, updating the places were variant arrays are defined could be easily forgotten.

Instead we can define the union type in terms of a co-located const array, both of which can be exported and used. Uppercase naming for the array is used to denote that it is a constant.

</aside>

### Example: Better approach ✅

- 😀 Single source of truth for union values
- 😀 Strongly typed, resilient to changes
- 🙂 Co-location of exported union and array

```tsx []
// types.ts
export const VARIANTS = ["accent", "negative", "warning", "info"] as const;
export type Variant = (typeof VARIANTS)[number];

// component.stories.tsx
export const ComponentVariants: Story = () => (
  <>
    {VARIANTS.map((variant: Variant) => (
      <Component key={variant} variant={variant} />
    ))}
  </>
);
```

<aside class="notes">
`as const` is required, as this prevents further modification of the array. Without it, the element type of the array would be widened to `string[]`.

An argument could be made that exporting the array from the `types.ts` is not ideal and should instead be exported from another file `constants.ts`. An exception can be made here, as there is value in co-locating the array with the type definition.

</aside>

### ✍️ Exercise: Reduce duplicate specifications of `BannerVariants` in the `Banner` component

<details>
  <summary>Click to expand exercise</summary>

1. Improve the `BannerVariants` type definition
2. Fix type errors
3. Improve naming

--

<iframe
  src="https://stackblitz.com/edit/vitejs-vite-3fci6h?ctl=1&embed=1&file=src%2Fbanner%2Ftypes.ts&hideExplorer=0&hideNavigation=1"
  style="width:100%; height: 600px; border:0; border-radius: 4px; overflow:hidden;"
  title="Workshop: String literal union example"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>
</details>

---

# Styles composition using `-styleSelector`

--

- **Pattern Type:** `Styled Components/SC specific`
- **References:** [sc-web-ui Development Guidelines](http://localhost:6006/?path=/docs/documentation-development-guidelines--docs#styling)
- **Code example:** [NotificationBadge](https://github.com/SafetyCulture/frontend-reactor/blob/6c7c28b64ad7c3c9f723126afa925a9f6f610249/sc-web-ui/packages/react/badge/notification-badge/notification-badge-styled.tsx#L62-L68)

### Description

When styling components, the permutations (and hence complexity) of possible styles can grow exponentially as props are added. The challenge is to compose styles in a way that is maintainable and scalable. Strong typing is a useful tool for this, and we can use `Record` typed object maps to declaratively define variants and eliminate the need for `if` and `switch` statements and mutable state.

### Example: Not scalable or type-safe ❌

- 🙁 Prone to error if types are updated
- 🙁 Untyped template strings can't catch typos or CSS mistakes
- 🙁 No separation of concerns

--

```tsx []
/**
 * Notification badge is intended to be used for displaying the number of items
 * requiring a user's attention. It can appear in the top corner of another element.
 */
export const StyledNotificationBadge = styled(NotificationBadge)`
  border-radius: ${(p) => p.theme._ui.radii.full};
  display: flex;
  align-items: center;
  justify-content: center;
  ${(p) =>
    p.size === "xSmall" && `min-width: ${pxToRem(8)}; height: ${pxToRem(8)};`}
  ${(p) =>
    p.size === "small" && `min-width: ${pxToRem(16)}; height: ${pxToRem(16)};`}
  ${(p) =>
    p.size === "medium" && `min-width: ${pxToRem(16)}; height: ${pxToRem(16)};`}
  ${(p) =>
    p.variant === "accent"
      ? `
      color: ${p.theme._ui.colors.accent.text.onDefault};
      background: ${p.theme._ui.colors.accent.border.default};`
      : `
      color: ${p.theme._ui.colors.negative.text.onDefault};
      background: ${p.theme._ui.colors.negative.border.default};`}
`;
```

### Example: Better approach ✅

- 😀 Strongly typed, resilient to changes
- 🙂 More verbose, but good separation of concerns
- 😐 May not be easy to 100% decouple related props
- 🙁 Provides no lint or type safety for shadowed CSS rules

--

```tsx []
import type {
  NotificationBadgeProps,
  NotificationBadgeSize,
  NotificationBadgeVariant,
} from "./types";
import { DEFAULT_SIZE } from "./types";

const sizeStyleSelector = ({
  size = DEFAULT_SIZE,
}: NotificationBadgeProps & WithThemeProps) => {
  const styles: Record<NotificationBadgeSize, FlattenSimpleInterpolation> = {
    xSmall: css`
      min-width: ${pxToRem(8)};
      height: ${pxToRem(8)};
    `,
    small: css`
      min-width: ${pxToRem(16)};
      height: ${pxToRem(16)};
    `,
    medium: css`
      min-width: ${pxToRem(20)};
      height: ${pxToRem(20)};
    `,
  };

  return styles[size];
};

const variantStyleSelector = ({
  variant,
  theme,
}: NotificationBadgeProps & WithThemeProps) => {
  const styles: Record<NotificationBadgeVariant, FlattenSimpleInterpolation> = {
    accent: css`
      color: ${theme._ui.colors.accent.text.onDefault};
      background: ${theme._ui.colors.accent.border.default};
    `,
    negative: css`
      color: ${theme._ui.colors.negative.text.onDefault};
      background: ${theme._ui.colors.negative.border.default};
    `,
  };

  return styles[variant];
};

/**
 * Notification badge is intended to be used for displaying the number of items
 * requiring a user's attention. It can appear in the top corner of another element.
 */
export const StyledNotificationBadge = styled(NotificationBadge)`
  border-radius: ${(p) => p.theme._ui.radii.full};
  display: flex;
  align-items: center;
  justify-content: center;
  ${sizeStyleSelector}
  ${variantStyleSelector}
`;
```

<aside class="notes">
This pattern encapsulates the logic for styling based on props and ensures that if new variants are added, removed or renamed, strong typing will help to prevent permutations from being missed. `FlattenSimpleInterpolation` and the `css` template literal tag also help with strong typing, helping to identify and prevent typos or mistakes in CSS rules. The `WithThemeProps` type allows us to access the theme object on the props which has colors values, spaces and other tokens.
</aside>

---

# Styled Selector Pattern

--

- **Pattern Type:** `Styled Components`
- **Prerequisites:** Styles composition using `styleSelector`
- **References:** [Styled Components docs](https://styled-components.com/docs/advanced#referring-to-other-components)
- **Code example:** [List](),

### Description

When using [Compound Components](#compound-component-pattern), it can be the case that a child-component requires styling based on logic or state that is known in the parent. For simple cases it may not be necessary to reach for more complex patterns such as [React Context](#react-context-for-compound-components), [Clone Element](#clone-element-pattern) or [Render Props](#render-props-pattern). Instead we can use the Styled Selector to target specific child (or recursive child) components with CSS rules.

### Example: Less simple approach ❌

See [Card](https://github.com/SafetyCulture/frontend-reactor/blob/96b525f192b41ae14c8b6600c2ad68a656775599/sc-web-ui/packages/react/card/card.tsx#L39-L47)

### Example: Simple approach ✅

```tsx []
// list-styled.tsx
import { StyledListItem as ListItem } from "./list-item-styled";

const variantSelector = ({
  variant = DEFAULT_VARIANT,
  theme,
}: ListProps & WithThemeProps) => {
  const { space, radii, colors } = theme._ui;
  const variantStylesMap: Record<Variant, FlattenSimpleInterpolation> = {
    seperated: css`
      gap: ${space.s3};
    `,
    rounded: css`
      gap: ${space.s1};
      ${ListItem}:first-child {
        border-top-left-radius: ${radii.md};
        border-top-right-radius: ${radii.md};
      }
      ${ListItem}:last-child {
        border-bottom-left-radius: ${radii.md};
        border-bottom-right-radius: ${radii.md};
      }
    `,
    stacked: css`
      ${ListItem}:not(:last-child) ${Divider} {
        border-bottom: solid 1px ${colors.surface.border.weakest};
      }
    `,
  };
  return variantStylesMap[variant];
};
```

---

# Separation of logic and styles

- **Pattern Type:** `Styled Components/SC specific`

### Description

When creating sc-web-ui components we like to follow the practice of creating "headless" components which are not concerned with any styling, and then wrap them in a styled wrapper (for example `StyledInput`).

```tsx []
// input-styled.tsx
import { Input } from "./input";

const StyledInput = styled(Input)`
  // styles...
`;
```

In the `index.ts` in the root of the component where we export it, we rename the `StyledInput` as `Input`.

```tsx []
// index.ts
export { StyledInput as Input } from "./input-styled";
```

---

# Expose native HTML attributes as props

### Description

By spreading props passed to our component by the consumer onto the underlying HTML element, we can allow the consumer to pass native HTML attributes to the component.

--

There are some considerations though as we have to be careful not to spread React props on to a native HTML element. 

--

There are two ways we can manage this, either by removing the props we don't want before spreading it onto the HTML element, or by stripping them out using a Styled Components validator in the styled component wrapper.

---

That's all for part 1! 🎉

---

# ForwardRef

- **Pattern Type:** `React`

### Description

---

# Polymorphic component pattern

- **Pattern Type:** `React`

Consider if the component needs to be rendered as a dynamic list of elements and components, or if a fixed set would suffice.

### Description

---

# Layouts component pattern

- **Pattern Type:** `React/SC specific`

### Description

---

# Compound component pattern

- **Pattern Type:** `React`
- **References:** [patterns.dev](https://www.patterns.dev/react/compound-pattern)

### Description

---

# Render props pattern

- **Pattern Type:** `React`
- **Prerequisites:** Dependency Injection, Children prop pattern, Clone element pattern

### Description

---

# Clone element pattern

- **Pattern Type:** `React`
- **Prerequisites:** Children prop pattern

### Description

---

# React Context for compound components

- **Pattern Type:** `React`

### Description

---
