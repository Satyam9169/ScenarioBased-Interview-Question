# Microfrontends and Monorepos — A Detailed Guide for React Developers

## 1. The big picture

Imagine an e-commerce platform with Product, Cart, Checkout, Profile, and Admin areas. As the application and organization grow, teams may need clearer ownership, shared code, faster builds, and more independent releases.

**Monorepo and microfrontend solve different problems:**

- **Monorepo:** Where and how multiple projects or packages are stored and managed.
- **Microfrontend:** How a frontend is divided into smaller modules that can be developed and potentially deployed independently.

They are not alternatives. A project can use both.

## 2. What is a monorepo?

A **monorepo** is a software-development strategy in which multiple applications, packages, or libraries are maintained in one Git repository.

### Separate repositories

```text
company-projects/
├── customer-app/   # Repository 1
├── admin-app/      # Repository 2
└── seller-app/     # Repository 3
```

### Monorepo

```text
ecommerce-platform/             # One Git repository
├── apps/
│   ├── customer/
│   ├── admin/
│   └── seller/
├── packages/
│   ├── ui-components/
│   ├── authentication/
│   ├── api-client/
│   └── utilities/
├── package.json
└── turbo.json
```

A monorepo can contain multiple applications. It does **not** mean there is only one application.

### Why use a monorepo?

- Reuse shared UI components and utilities.
- Coordinate changes across apps in a single pull request.
- Share linting, formatting, TypeScript, and testing configuration.
- Manage dependencies and related projects in one place.
- Improve build performance with caching and dependency-aware task execution.

### Example shared React component

`packages/ui/src/Button.jsx`

```jsx
export default function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}
```

A consuming application might use it like this, assuming the workspace package exports and aliases are configured:

```jsx
import Button from "@company/ui/Button";

export default function CustomerApp() {
  return (
    <div>
      <h1>Customer Dashboard</h1>
      <Button onClick={() => alert("Clicked")}>
        View Products
      </Button>
    </div>
  );
}
```

A second application can import the same package. Whether a change becomes available immediately depends on how the package is built, consumed, and deployed.

### Monorepo tools

- **Git:** Version control and repository history.
- **npm, pnpm, or Yarn workspaces:** Manage projects and package dependencies in a repository.
- **Turborepo:** Coordinates tasks and caches build/test results.
- **Nx:** Provides project graph, task orchestration, caching, and workspace tooling.

Turborepo and Nx are optional; a monorepo can exist without them.

### Monorepo trade-offs

**Advantages**
- Shared code and consistent tooling.
- Easier cross-project refactoring.
- Visibility into related apps and packages.
- Task caching can avoid unnecessary repeated work.

**Disadvantages**
- Repository can become large and complex.
- Poorly configured pipelines may run too many builds and tests.
- Dependency versions may conflict.
- Repository-level access control can be harder for teams that need strict isolation.
- A breaking change in a shared package can affect many consumers.

**Use a monorepo when** multiple projects share code or tooling and coordinated changes are valuable. Multiple repositories can still be better when access boundaries, independent governance, or low coupling matter more.

## 3. What is a microfrontend?

A **microfrontend** is an architectural approach that divides a large frontend into smaller, business-focused modules that can be developed and potentially deployed independently.

For example:

```text
                 E-commerce Website
                         |
                    Host / Shell
                         |
          ┌──────────────┼──────────────┐
          |              |              |
       Product           Cart         Checkout
          |              |              |
     Product team     Cart team     Checkout team
```

Each team can own its module's code, tests, and release process. The host or shell integrates the modules into one user experience.

Independent deployment is a goal, not an automatic guarantee. The build, deployment, versioning, and runtime integration must support it.

### A simple React example

A Product Microfrontend might render:

```jsx
export default function ProductApp() {
  return (
    <section>
      <h2>Products</h2>
      <p>Product listing goes here.</p>
    </section>
  );
}
```

A Cart Microfrontend might render:

```jsx
export default function CartApp() {
  return (
    <section>
      <h2>Shopping Cart</h2>
      <p>Cart items go here.</p>
    </section>
  );
}
```

A shell may compose them conceptually:

```jsx
export default function App() {
  return (
    <>
      <header>My E-commerce Website</header>
      <main>
        <ProductSection />
        <CartSection />
      </main>
    </>
  );
}
```

`ProductSection` and `CartSection` represent integration points. This example shows composition only; it does not implement runtime loading or independent deployment. Real projects use a mechanism such as Module Federation, single-spa, web components, or route-based composition.

## 4. Module Federation

**Webpack Module Federation** lets separately built applications expose and consume modules at runtime. It is one way to implement microfrontends; it is not synonymous with the microfrontend architecture.

A simplified remote configuration can look like this:

```js
new ModuleFederationPlugin({
  name: "product",
  filename: "remoteEntry.js",
  exposes: {
    "./ProductApp": "./src/ProductApp.jsx",
  },
  shared: ["react", "react-dom"],
});
```

The host must also configure the remote and import the exposed module. Exact configuration depends on the bundler, versions, and framework.

Important production concerns include:
- Compatible React and shared dependency versions.
- Remote availability and graceful failure handling.
- Cache invalidation and versioning.
- Rollbacks and deployment coordination.
- Monitoring, testing, and security.

## 5. How microfrontends communicate

### A. Browser custom events

The Product module can dispatch an event:

```jsx
function AddToCartButton({ product }) {
  function handleAdd() {
    window.dispatchEvent(
      new CustomEvent("cart:add", { detail: product })
    );
  }

  return <button onClick={handleAdd}>Add to Cart</button>;
}
```

The Cart module can listen:

```jsx
import { useEffect, useState } from "react";

export default function CartApp() {
  const [items, setItems] = useState([]);

  useEffect(() => {
    const handleAdd = (event) => {
      setItems((current) => [...current, event.detail]);
    };

    window.addEventListener("cart:add", handleAdd);
    return () => window.removeEventListener("cart:add", handleAdd);
  }, []);

  return <h2>Cart items: {items.length}</h2>;
}
```

This is a teaching example. Production systems should validate payloads, define stable event contracts, and avoid trusting browser-supplied prices or sensitive data.

### B. Shared state

A shared store can coordinate closely integrated modules, but a single global Redux store can increase coupling. Agree on ownership, contracts, and compatibility before sharing state.

### C. Routing

```text
/products  → Product module
/cart      → Cart module
/checkout  → Checkout module
/profile   → Profile module
```

The shell or routing layer decides which module is displayed.

### D. Backend APIs

Modules can use a backend as the source of truth. For example, Product adds an item through an API, and Cart fetches the updated cart. This often reduces direct frontend coupling.

| Need | Possible approach |
|---|---|
| Notify a module about a UI event | Custom events |
| Coordinate tightly coupled UI state | Shared state |
| Navigate between business areas | URL and routing |
| Keep business data consistent | Backend APIs |

There is no universally best communication strategy. Choose based on consistency, coupling, ownership, and runtime boundaries.

## 6. Authentication and authorization

A shell may coordinate the login experience and provide a common authentication mechanism. Each module still needs to use that mechanism correctly when calling protected APIs.

- **Authentication:** Who is the user?
- **Authorization:** What is the user allowed to do?

Frontend route protection improves user experience but is not a security boundary. The backend must validate authentication and enforce authorization for protected data and actions. Plan for expired sessions, logout, token safety, and consistent error handling.

## 7. Microfrontend advantages and disadvantages

**Advantages**
- Clear business-domain and team ownership.
- Potentially independent releases.
- Smaller team-focused codebases.
- Technology flexibility when there is a genuine need.
- Less coordination for changes that stay within a module's contract.

**Disadvantages**
- More integration and operational complexity.
- Potentially more JavaScript and runtime overhead.
- Shared dependency and React-version issues.
- Inconsistent UI if teams do not share a design system.
- Harder debugging across host, remotes, and contracts.
- More integration testing and monitoring work.

Use microfrontends when independent ownership and releases solve real organizational problems. A well-structured single React application is often simpler for a small team or product.

## 8. Monorepo vs microfrontend

| Feature | Monorepo | Microfrontend |
|---|---|---|
| Type | Repository-management strategy | Frontend architecture |
| Main question | Where is code managed? | How is the frontend divided and integrated? |
| Focus | Packages, source, tooling, workflows | Module boundaries, composition, deployment |
| Git repositories | Usually one | Can be one or many |
| Independent deployment | Possible, but not implied | Common goal, but requires supporting infrastructure |
| Shared code | Workspace packages are common | Shared packages or runtime contracts |
| Tools | pnpm, npm/Yarn workspaces, Turborepo, Nx | Module Federation, single-spa, web components |

**Memory trick:** Monorepo = where code lives. Microfrontend = how the frontend is split and assembled.

## 9. Using both together

A company can keep multiple independently deployable microfrontends in one monorepo:

```text
ecommerce-platform/
├── apps/
│   ├── shell/
│   ├── product-mfe/
│   ├── cart-mfe/
│   ├── checkout-mfe/
│   └── profile-mfe/
├── packages/
│   ├── ui-components/
│   ├── api-client/
│   ├── authentication/
│   └── design-tokens/
├── package.json
├── pnpm-workspace.yaml
└── turbo.json
```

The monorepo provides one place to manage source and shared packages. The microfrontend architecture defines module boundaries and runtime composition. Deployment pipelines can still build and release modules independently.

### Example deployment flow

1. A developer updates `apps/product-mfe`.
2. A pull request runs the relevant tests and checks.
3. The change is merged.
4. The pipeline builds and deploys the Product remote.
5. The host loads the version configured for that remote.

The Cart and Checkout modules need not be redeployed if the change does not affect them and the architecture supports independent releases.

## 10. Shared component changes

Suppose three microfrontends consume a shared Button package. Changing its API can break all three.

Ways to manage this risk:
1. Prefer backward-compatible changes when practical.
2. Test affected consumers.
3. Define package contracts and versioning.
4. Coordinate breaking changes.
5. Use dependency-aware build and test pipelines.

A monorepo makes coordinated changes easier; it does not eliminate compatibility problems. Build-time consumers usually need to rebuild to include the updated implementation.

## 11. How to choose an architecture

For a large e-commerce platform with multiple teams:

1. **Assess repository needs.** If teams share code and benefit from coordinated changes, consider a monorepo.
2. **Assess release independence.** If release coordination is a serious bottleneck, consider microfrontends. Otherwise, modular React may be enough.
3. **Define business boundaries.** Give Product, Cart, Checkout, and Profile clear responsibilities.
4. **Define shared packages.** Keep UI, API clients, types, and utilities focused; excessive sharing can recreate tight coupling.
5. **Design deployment.** Specify builds, tests, runtime configuration, versioning, rollback, and remote-failure behavior.
6. **Measure the outcome.** Track performance, bundle size, user experience, and release reliability.

Do not choose microfrontends simply because the app is large. Choose them when independent ownership and deployment justify the extra complexity.

## 12. Senior React interview questions

1. Your company has five React applications and a shared UI library. Teams need independent releases. Would you choose a monorepo, microfrontends, or both? Explain your architecture.
2. A microfrontend fails after deployment because its React dependency is incompatible with the host. How would you investigate and prevent this?
3. A Product remote was deployed, but the host still loads the old version. How would you investigate versioning and caching?
4. Product adds an item to the cart, but the Cart module does not update. How would you design communication without excessive coupling?
5. A shared UI package update breaks three applications. How would you improve testing, versioning, and release controls?
6. A two-developer team wants to split a small React application into microfrontends. What factors would guide your decision?

## 13. Suggested learning sequence

1. React architecture: components, routing, state, modular design.
2. Monorepo fundamentals: workspaces and shared packages.
3. Tooling: pnpm workspaces and Turborepo or Nx.
4. Microfrontend concepts: host, remote, composition, and ownership.
5. Module Federation or another integration strategy.
6. Routing, custom events, state contracts, and APIs.
7. Deployment, caching, runtime configuration, and rollback.
8. Production concerns: security, testing, observability, and performance.

## 14. Official learning resources

- Webpack Module Federation: https://webpack.js.org/concepts/module-federation/
- Turborepo: https://turborepo.com/docs
- Nx: https://nx.dev/docs
- pnpm Workspaces: https://pnpm.io/workspaces
- single-spa: https://single-spa.js.org/docs/getting-started-overview/

## 15. Final revision

1. **Monorepo:** Multiple projects or packages maintained in one Git repository.
2. **Microfrontend:** A frontend architecture divided into smaller, business-focused modules.
3. **Module Federation:** One technology for exposing and consuming modules at runtime.
4. **Monorepo + Microfrontend:** One repository can contain independently deployable frontend modules and shared packages.
5. **Choose based on needs:** Team ownership, release requirements, complexity, security, and performance.

**Recommended hands-on exercise:** Create a pnpm workspace with a React shell and two small apps. First make them ordinary workspace applications; then explore a microfrontend integration approach such as Module Federation. This makes the difference between repository organization and runtime composition concrete.
