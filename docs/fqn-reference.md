# FQN Reference

## Grammar

Fully Qualified Names (FQNs) address components and their sections across files.

```
fqn        = [project ":"] scope-path ["#" fragment] ["@" version]
scope-path = segment (":" segment)*
fragment   = aspect-or-section [":" sub-path]
segment    = kebab-case-identifier
```

## Parts

| Part | Required | Description | Example |
|------|----------|-------------|---------|
| `project` | No | Project namespace | `acme:` |
| `scope-path` | Yes | Colon-separated component path | `cloud:arm:my-service` |
| `fragment` | No | Section or aspect target | `#api`, `#impl` |
| `version` | No | Version constraint | `@1.0.0` |

## Fragment Types

| Fragment | Aliases | Description |
|----------|---------|-------------|
| `#api` | `#interface` | Public interface section |
| `#impl` | `#implementation` | Internal implementation (own scope only) |
| `#infra` | — | Infrastructure aspect |
| `#data` | — | Data/persistence aspect |
| `#security` | — | Security aspect |
| `#monitoring` | — | Monitoring aspect |

### Sub-path Fragments

Fragments can target specific elements within a section:

```
my-service#api:users-add        → the users-add endpoint in the interface
my-service#api:events:user-created  → specific event
```

## Examples

### Simple Component

```
user-service
```

References `user-service` in the current project scope.

### With Project Scope

```
acme:cloud:arm:my-service
```

Full path: project `acme`, scope `cloud:arm`, component `my-service`.

### Interface Reference

```
user-service#api
```

Only the public interface of `user-service`.

### Specific Endpoint

```
user-service#api:create-user
```

The `create-user` endpoint within the interface.

### Implementation (Private)

```
user-service#impl
```

Internal implementation — only visible to `user-service` itself.

### Aspect Reference

```
user-service#infra
```

The infrastructure aspect view.

### Versioned Reference

```
user-service@0.3.0
```

Pin to a specific version.

## Resolution Rules

1. **Relative** — bare names resolve within the current project scope
2. **Absolute** — project-prefixed names resolve globally
3. **Fragment filtering** — `#api` projects only the public interface; implementation details are stripped
4. **Facet filtering** — `facet="data"` in a dependency further narrows what is projected

## Usage in CDML

FQNs appear in component definitions and dependency declarations:

```xml
<!-- Component declaration -->
<component name="my-service" fqn="acme:cloud:arm:my-service">

<!-- Dependency reference -->
<requires>
  <dependency fqn="redis#api" usage="cache" />
  <dependency fqn="acme:cloud:arm:other-service#api:users-add" />
</requires>

<!-- Context manifest -->
<scope path="my-service > implementation > user-controller">
  <interfaces>
    <fulfills fqn="my-service#api:create-user" />
  </interfaces>
</scope>
```
