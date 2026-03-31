![Patterns.dev Skills](https://res.cloudinary.com/ddxwdqwkr/image/upload/v1770936713/patterns.dev/patterns-cover.webp)

# Patterns.dev Skills

[Agent Skills](https://agentskills.io) for JavaScript, React, and Vue development. From [patterns.dev](https://patterns.dev).

58 individual skills. Install only the patterns you need.

## Quick Start

**By technology** — install skills in a stack:

```bash
npx skills add PatternsDev/skills/javascript
npx skills add PatternsDev/skills/react
npx skills add PatternsDev/skills/vue
```

**By skill name** — install only the specific patterns you want:

```bash
npx skills add PatternsDev/skills --skill hooks-pattern
npx skills add PatternsDev/skills --skill singleton-pattern
npx skills add PatternsDev/skills --skill composables
npx skills add PatternsDev/skills --skill ai-ui-patterns
```

> Individual `--skill` values use the skill folder name (e.g. `hooks-pattern`, `ai-ui-patterns`), not the path (`react/hooks-pattern`).

## Manual Installation

Clone the repo and copy the skill directories you want:

```bash
cp -r react/hooks-pattern ~/.cursor/skills/
cp -r javascript/singleton-pattern ~/.cursor/skills/
cp -r vue/composables ~/.cursor/skills/
```

Works with `.cursor/skills/`, `.claude/skills/`, or `.codex/skills/`.

## [JavaScript Skills](javascript/)

| Skill | Type | Description |
|-------|------|-------------|
| [`singleton-pattern`](javascript/singleton-pattern/SKILL.md) | Design | Share a single global instance throughout our application |
| [`observer-pattern`](javascript/observer-pattern/SKILL.md) | Design | Use observables to notify subscribers when an event occurs |
| [`proxy-pattern`](javascript/proxy-pattern/SKILL.md) | Design | Intercept and control interactions to target objects |
| [`prototype-pattern`](javascript/prototype-pattern/SKILL.md) | Design | Share properties among many objects of the same type |
| [`module-pattern`](javascript/module-pattern/SKILL.md) | Design | Split up your code into smaller, reusable pieces |
| [`mixin-pattern`](javascript/mixin-pattern/SKILL.md) | Design | Add functionality to objects or classes without inheritance |
| [`mediator-pattern`](javascript/mediator-pattern/SKILL.md) | Design | Use a central mediator object to handle communication between components |
| [`flyweight-pattern`](javascript/flyweight-pattern/SKILL.md) | Design | Reuse existing instances when working with identical objects |
| [`factory-pattern`](javascript/factory-pattern/SKILL.md) | Design | Use a factory function in order to create objects |
| [`command-pattern`](javascript/command-pattern/SKILL.md) | Design | Decouple methods that execute tasks by sending commands to a commander |
| [`provider-pattern`](javascript/provider-pattern/SKILL.md) | Design | Make data available to multiple child components |
| [`loading-sequence`](javascript/loading-sequence/SKILL.md) | Performance | Optimize your loading sequence to improve how quickly your app is usable |
| [`static-import`](javascript/static-import/SKILL.md) | Performance | Import code that has been exported by another module |
| [`dynamic-import`](javascript/dynamic-import/SKILL.md) | Performance | Import parts of your code on demand |
| [`import-on-visibility`](javascript/import-on-visibility/SKILL.md) | Performance | Load non-critical components when they are visible in the viewport |
| [`import-on-interaction`](javascript/import-on-interaction/SKILL.md) | Performance | Load non-critical resources when a user interacts with UI requiring it |
| [`route-based`](javascript/route-based/SKILL.md) | Performance | Dynamically load components based on the current route |
| [`bundle-splitting`](javascript/bundle-splitting/SKILL.md) | Performance | Split your code into small, reusable pieces |
| [`prpl`](javascript/prpl/SKILL.md) | Performance | Optimize initial load through precaching, lazy loading, and minimizing roundtrips |
| [`tree-shaking`](javascript/tree-shaking/SKILL.md) | Performance | Reduce the bundle size by eliminating dead code |
| [`preload`](javascript/preload/SKILL.md) | Performance | Inform the browser of critical resources before they are discovered |
| [`prefetch`](javascript/prefetch/SKILL.md) | Performance | Fetch and cache resources that may be requested some time soon |
| [`third-party`](javascript/third-party/SKILL.md) | Performance | Reduce the performance impact third-party scripts have on your site |
| [`virtual-lists`](javascript/virtual-lists/SKILL.md) | Performance | Optimize list performance with list virtualization |
| [`compression`](javascript/compression/SKILL.md) | Performance | Reduce the time needed to transfer scripts over the network |
| [`islands-architecture`](javascript/islands-architecture/SKILL.md) | Rendering | Small, focused chunks of interactivity within server-rendered web pages |
| [`view-transitions`](javascript/view-transitions/SKILL.md) | Rendering | Animate page transitions using the View Transitions API |

## [React Skills](react/)

| Skill | Type | Description |
|-------|------|-------------|
| [`hooks-pattern`](react/hooks-pattern/SKILL.md) | Design | Use functions to reuse stateful logic among multiple components |
| [`hoc-pattern`](react/hoc-pattern/SKILL.md) | Design | Pass reusable logic down as props to components |
| [`compound-pattern`](react/compound-pattern/SKILL.md) | Design | Create multiple components that work together to perform a single task |
| [`render-props-pattern`](react/render-props-pattern/SKILL.md) | Design | Pass JSX elements to components through props |
| [`presentational-container-pattern`](react/presentational-container-pattern/SKILL.md) | Design | Enforce separation of concerns by separating the view from the application logic |
| [`ai-ui-patterns`](react/ai-ui-patterns/SKILL.md) | Design | Design patterns for building AI-powered interfaces like chatbots and assistants |
| [`react-2026`](react/react-2026/SKILL.md) | Design | Comprehensive guide to building React apps with a modern 2026 stack |
| [`client-side-rendering`](react/client-side-rendering/SKILL.md) | Rendering | Render your application's UI on the client |
| [`server-side-rendering`](react/server-side-rendering/SKILL.md) | Rendering | Generate HTML to be rendered on the server in response to a user request |
| [`static-rendering`](react/static-rendering/SKILL.md) | Rendering | Deliver pre-rendered HTML content that was generated when the site was built |
| [`incremental-static-rendering`](react/incremental-static-rendering/SKILL.md) | Rendering | Update static content after you have built your site |
| [`streaming-ssr`](react/streaming-ssr/SKILL.md) | Rendering | Stream HTML to the client as it is generated on the server |
| [`progressive-hydration`](react/progressive-hydration/SKILL.md) | Rendering | Delay loading JavaScript for less important parts of the page |
| [`react-server-components`](react/react-server-components/SKILL.md) | Rendering | Server Components compliment SSR, rendering without adding to the JS bundle |
| [`react-selective-hydration`](react/react-selective-hydration/SKILL.md) | Rendering | Combine streaming server-side rendering with selective hydration |

## [Vue Skills](vue/)

| Skill | Type | Description |
|-------|------|-------------|
| [`components`](vue/components/SKILL.md) | Design | Self-contained modules that couple markup, logic, and styles |
| [`composables`](vue/composables/SKILL.md) | Design | Functions to encapsulate and reuse stateful logic among multiple components |
| [`script-setup`](vue/script-setup/SKILL.md) | Design | Compile-time syntactic sugar for using the Composition API |
| [`state-management`](vue/state-management/SKILL.md) | Design | Manage application level state between components |
| [`provide-inject`](vue/provide-inject/SKILL.md) | Design | Have nested components access data without using props |
| [`dynamic-components`](vue/dynamic-components/SKILL.md) | Design | Dynamically switch between components with the special component element |
| [`async-components`](vue/async-components/SKILL.md) | Performance | Optimize web app performance by asynchronously loading components |
| [`render-functions`](vue/render-functions/SKILL.md) | Design | Create component templates with programmatic JavaScript |
| [`renderless-components`](vue/renderless-components/SKILL.md) | Design | Components that don't render their own markup |
| [`container-presentational`](vue/container-presentational/SKILL.md) | Design | Enforce separation of concerns by separating the view from the application logic |
| [`data-provider`](vue/data-provider/SKILL.md) | Design | Utilize renderless components for managing and providing data |

## How skills differ from patterns.dev documentation

The [patterns.dev](https://patterns.dev) website includes interactive CodeSandbox embeds, animated videos, diagrams, and visual walkthroughs. The skills here are **agent-optimized**, stripped down to just prose and code blocks. No images, no iframes, no visual assets.

For the full visual learning experience, visit [patterns.dev](https://patterns.dev).

If you want a more explicit comparison of positioning and scope, see [KEY-DIFFS.md](KEY-DIFFS.md) for notes on how these skills differ from Vercel's skills approach.

## Resources

- [github.com/PatternsDev/skills](https://github.com/PatternsDev/skills) — source repository
- [patterns.dev](https://patterns.dev) — the full pattern library with interactive examples
- [Agent Skills Specification](https://agentskills.io/specification)
- [Agent Skills Overview](https://agentskills.io)
