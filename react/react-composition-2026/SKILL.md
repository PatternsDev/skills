---
name: react-composition-2026
description: Modern React composition patterns for 2025/2026. Use when designing component APIs, building shared UI libraries, or refactoring prop-heavy components.
context: fork
allowed-tools: Read, Grep, Glob
paths:
  - "**/*.tsx"
  - "**/*.jsx"
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
related_skills:
  - "hooks-pattern"
  - "hoc-pattern"
---

# Modern React Composition Patterns

Composition patterns for building flexible, maintainable React components that scale. These patterns replace boolean-prop proliferation, rigid component APIs, and tangled state with composable, explicit designs.

## When to Use

Reference these patterns when:
- A component has more than 3-4 boolean props controlling its behavior
- Building reusable UI components or a shared component library
- Refactoring components that are difficult to extend
- Designing component APIs that other teams will consume
- Reviewing component architecture for flexibility and maintainability

## Instructions

- Apply these patterns during component design, code generation, and review. When you see boolean prop accumulation or rigid component APIs, suggest the appropriate composition pattern.

## Details

### Overview

The core principle: **composition over configuration**. Instead of adding boolean props and conditional branches to handle every variant, compose smaller, focused components together. This makes components easier to understand, test, and extend — for both humans and AI agents.

---

### 1. Replace Boolean Props with Composition

**Impact: HIGH** — Prevents combinatorial explosion and makes intent explicit.

Boolean props multiply complexity: 4 booleans = 16 possible states, most of which are untested. Replace them with composable children.

**Avoid — boolean prop accumulation:**

```tsx
<Card
  showHeader
  showFooter
  collapsible
  bordered
  withShadow
  headerAction="close"
  size="large"
/>
```

**Prefer — explicit composition:**

```tsx
<Card variant="bordered" shadow="md">
  <Card.Header>
    <h3>Title</h3>
    <Card.CloseButton />
  </Card.Header>
  <Card.Body collapsible>
    <p>Content here</p>
  </Card.Body>
  <Card.Footer>
    <Button>Save</Button>
  </Card.Footer>
</Card>
```

Each piece is explicit, testable, and independently optional.

---

### 2. Build Compound Components with Context

**Impact: HIGH** — Shared implicit state without prop drilling.

Compound components are a group of components that work together, sharing state through context rather than props. The parent owns the state; children consume it.

**Avoid — parent manages everything through props:**

```tsx
<Select
  options={options}
  value={value}
  onChange={onChange}
  renderOption={(opt) => <span>{opt.icon} {opt.label}</span>}
  renderSelected={(opt) => <b>{opt.label}</b>}
  placeholder="Choose..."
  clearable
  searchable
  maxHeight={300}
/>
```

**Prefer — compound components:**

```tsx
const SelectContext = createContext<SelectState | null>(null)

function Select({ children, value, onChange }: SelectProps) {
  const [open, setOpen] = useState(false)
  const ctx = useMemo(() => ({ value, onChange, open, setOpen }), [value, onChange, open])

  return (
    <SelectContext.Provider value={ctx}>
      <div className="select-root">{children}</div>
    </SelectContext.Provider>
  )
}

function Trigger({ children }: { children: React.ReactNode }) {
  const { open, setOpen } = useSelectContext()
  return <button onClick={() => setOpen(!open)}>{children}</button>
}

function Options({ children }: { children: React.ReactNode }) {
  const { open } = useSelectContext()
  if (!open) return null
  return <ul role="listbox">{children}</ul>
}

function Option({ value, children }: OptionProps) {
  const { value: selected, onChange, setOpen } = useSelectContext()
  return (
    <li
      role="option"
      aria-selected={value === selected}
      onClick={() => { onChange(value); setOpen(false) }}
    >
      {children}
    </li>
  )
}

Select.Trigger = Trigger
Select.Options = Options
Select.Option = Option
```

**Usage:**

```tsx
<Select value={color} onChange={setColor}>
  <Select.Trigger>Pick a color</Select.Trigger>
  <Select.Options>
    <Select.Option value="red">Red</Select.Option>
    <Select.Option value="blue">Blue</Select.Option>
  </Select.Options>
</Select>
```

---

### 3. Create Explicit Variant Components

**Impact: MEDIUM** — Makes each mode a clear, focused component.

When a component has distinct "modes" (dialog vs drawer, inline vs modal, card vs list-item), create explicit variant components instead of toggling with props.

**Avoid — one component with mode props:**

```tsx
function MediaDisplay({ type, src, title, showControls, autoPlay, loop }: Props) {
  if (type === 'video') {
    return <video src={src} controls={showControls} autoPlay={autoPlay} loop={loop} />
  }
  if (type === 'audio') {
    return <audio src={src} controls={showControls} />
  }
  return <img src={src} alt={title} />
}
```

**Prefer — explicit variants:**

```tsx
function VideoPlayer({ src, controls, autoPlay, loop }: VideoProps) {
  return <video src={src} controls={controls} autoPlay={autoPlay} loop={loop} />
}

function AudioPlayer({ src, controls }: AudioProps) {
  return <audio src={src} controls={controls} />
}

function Image({ src, alt }: ImageProps) {
  return <img src={src} alt={alt} />
}
```

Each variant has exactly the props it needs — no impossible states, no unused props.

---

### 4. Use Children Over Render Props for Composition

**Impact: MEDIUM** — Simpler API, better readability.

Render props (`renderHeader`, `renderItem`) were essential before hooks, but today `children` provides cleaner composition for most cases.

**Avoid — render prop proliferation:**

```tsx
<DataTable
  data={users}
  renderHeader={() => <h2>Users</h2>}
  renderRow={(user) => <UserRow user={user} />}
  renderEmpty={() => <EmptyState />}
  renderFooter={() => <Pagination />}
/>
```

**Prefer — children composition:**

```tsx
<DataTable data={users}>
  <DataTable.Header>
    <h2>Users</h2>
  </DataTable.Header>
  <DataTable.Body>
    {users.map(user => <UserRow key={user.id} user={user} />)}
  </DataTable.Body>
  <DataTable.Empty>
    <EmptyState />
  </DataTable.Empty>
  <DataTable.Footer>
    <Pagination />
  </DataTable.Footer>
</DataTable>
```

Reserve render props for cases where the parent needs to provide data to the renderer (e.g., virtualized list items).

---

### 5. Decouple State Implementation from UI

**Impact: MEDIUM** — Swap state management without changing components.

Define a generic interface for your state shape (value, actions, metadata), then let providers implement it. Components consume the interface, not the implementation.

**Define the interface:**

```typescript
interface CounterState {
  count: number
  increment: () => void
  decrement: () => void
  isLoading: boolean
}

const CounterContext = createContext<CounterState | null>(null)

function useCounter() {
  const ctx = useContext(CounterContext)
  if (!ctx) throw new Error('useCounter must be used within a CounterProvider')
  return ctx
}
```

**Implement with local state:**

```tsx
function LocalCounterProvider({ children }: { children: React.ReactNode }) {
  const [count, setCount] = useState(0)
  const value = useMemo(() => ({
    count,
    increment: () => setCount(c => c + 1),
    decrement: () => setCount(c => c - 1),
    isLoading: false,
  }), [count])
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>
}
```

**Swap to API-backed state without changing consumers:**

```tsx
function ApiCounterProvider({ children }: { children: React.ReactNode }) {
  const { data, mutate } = useSWR('/api/counter', fetcher)
  const value = useMemo(() => ({
    count: data?.count ?? 0,
    increment: () => mutate(patch('/api/counter', { delta: 1 })),
    decrement: () => mutate(patch('/api/counter', { delta: -1 })),
    isLoading: !data,
  }), [data, mutate])
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>
}
```

The `useCounter()` consumers never change.

---

### 6. Lift State to Provider Components

**Impact: MEDIUM** — Enables sibling communication without prop threading.

When two sibling components need shared state, lift it into a provider rather than threading callbacks through the parent.

**Avoid — parent threads state to siblings:**

```tsx
function Page() {
  const [selected, setSelected] = useState<string | null>(null)
  return (
    <div>
      <Sidebar selected={selected} onSelect={setSelected} />
      <Detail selected={selected} />
    </div>
  )
}
```

**Prefer — provider manages shared state:**

```tsx
function SelectionProvider({ children }: { children: React.ReactNode }) {
  const [selected, setSelected] = useState<string | null>(null)
  return (
    <SelectionContext.Provider value={{ selected, setSelected }}>
      {children}
    </SelectionContext.Provider>
  )
}

function Page() {
  return (
    <SelectionProvider>
      <Sidebar />
      <Detail />
    </SelectionProvider>
  )
}
```

Both `Sidebar` and `Detail` consume `useSelection()` directly.

---

### 7. Use Polymorphic `as` Props for Flexible Elements

**Impact: MEDIUM** — One component, any underlying element or component.

The `as` prop pattern lets consumers control the rendered element while keeping your component's styles and behavior.

```tsx
type BoxProps<C extends React.ElementType = 'div'> = {
  as?: C
  children: React.ReactNode
} & Omit<React.ComponentPropsWithoutRef<C>, 'as' | 'children'>

function Box<C extends React.ElementType = 'div'>({
  as,
  children,
  ...props
}: BoxProps<C>) {
  const Component = as || 'div'
  return <Component {...props}>{children}</Component>
}
```

**Usage:**

```tsx
<Box>Default div</Box>
<Box as="section">A section</Box>
<Box as="a" href="/about">A link</Box>
<Box as={Link} to="/about">Router link</Box>
```

---

### 8. React 19: Drop `forwardRef`, Use `ref` as a Prop

**Impact: MEDIUM** — Simpler component definitions.

React 19 passes `ref` as a regular prop. No more `forwardRef` wrapper.

**React 18 (deprecated pattern):**

```tsx
const Input = forwardRef<HTMLInputElement, InputProps>(function Input(props, ref) {
  return <input ref={ref} {...props} />
})
```

**React 19:**

```tsx
function Input({ ref, ...props }: InputProps & { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />
}
```

Similarly, `use()` can read either promises or context and can be called conditionally:

```tsx
import { use } from 'react'

function Panel({ themePromise }: { themePromise: Promise<Theme> }) {
  const theme = use(themePromise)  // unwraps promise
  const user = use(UserContext)    // conditional context read
  return <div className={theme.bg}>{user.name}</div>
}
```

---

### 9. Slot Pattern for Layout Components

**Impact: MEDIUM** — Named insertion points without render props.

For layout components with multiple content areas, use a slot pattern based on child type detection or named sub-components.

```tsx
function AppLayout({ children }: { children: React.ReactNode }) {
  const slots = React.Children.toArray(children)
  const header = slots.find(
    (child): child is React.ReactElement => React.isValidElement(child) && child.type === AppLayout.Header
  )
  const content = slots.filter(
    (child) => !React.isValidElement(child) || child.type !== AppLayout.Header
  )

  return (
    <div className="app-layout">
      <header>{header}</header>
      <main>{content}</main>
    </div>
  )
}

AppLayout.Header = function Header({ children }: { children: React.ReactNode }) {
  return <>{children}</>
}
```

**Usage:**

```tsx
<AppLayout>
  <AppLayout.Header>
    <Logo />
    <Nav />
  </AppLayout.Header>
  <Dashboard />
</AppLayout>
```

---

### 10. Headless Components for Maximum Flexibility

**Impact: HIGH** — Logic without opinions about rendering.

Headless components provide behavior (state, keyboard handling, ARIA attributes) without any markup. Consumers supply the rendering.

```tsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial)
  const toggle = useCallback(() => setOn(o => !o), [])
  const buttonProps = {
    'aria-pressed': on,
    onClick: toggle,
    role: 'switch' as const,
  }
  return { on, toggle, buttonProps }
}
```

**Usage — consumer controls all rendering:**

```tsx
function DarkModeSwitch() {
  const { on, buttonProps } = useToggle(false)
  return (
    <button {...buttonProps} className={on ? 'dark' : 'light'}>
      {on ? 'Dark' : 'Light'} Mode
    </button>
  )
}
```

Libraries like Radix UI, Headless UI, and React Aria follow this pattern. Prefer them over fully-styled component libraries when you need design flexibility.

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — composition guidance for the broader React community.
