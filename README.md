# ~guilhermeledes.github.io~

Public technical documentation for Guilherme Ledes's current portfolio platform.

The live site is served at [guilhermeledes.dev](https://guilhermeledes.dev/). This repository remains public as the GitHub Pages redirect entrypoint and as a lightweight public architecture reference for the live portfolio system.

## Overview

The portfolio is a content-driven web application designed to present Guilherme Ledes's professional profile, selected roles, curated project case studies, and a grounded AI-assisted Q&A experience.

The platform is intentionally small:

- one web application
- one reverse proxy
- curated content stored in-repo
- no database
- no background workers
- no persistent user accounts

The design goal is to keep the system operationally simple while still supporting:

- recruiter-facing landing content
- project detail pages
- CV presentation
- SEO metadata
- grounded AI answers over curated portfolio content
- containerized deployment on a VPS

The visual design direction for the portfolio was created with [Lovable](https://lovable.dev/), then implemented in the application codebase.

## High-Level Design

At a high level, the system is a monolithic `Next.js` application behind a reverse proxy. Content is authored as structured files, loaded at build/runtime by the app, and reused across the public pages and the grounded chat experience.

```mermaid
flowchart LR
    U["Visitor Browser"] --> P["Reverse Proxy (Caddy)"]
    P --> A["Next.js App Router Monolith"]
    A --> C["Curated Content Collections"]
    A --> O["OpenRouter API"]
    A --> H["Healthcheck Endpoint"]
```

## Main Runtime Components

### 1. Web application

The application is built as a `Next.js` App Router monolith. It is responsible for:

- rendering the public pages
- loading and validating authored content
- generating SEO metadata
- exposing the chat API
- brokering chat requests to the configured AI preset

### 2. Content layer

The portfolio content is curated rather than CMS-driven. The source model is organized into collections such as:

- `profile`
- `roles`
- `projects`

This content powers:

- the home page
- project detail pages
- the CV route
- the grounded portfolio narratives exposed by the AI experience

### 3. AI provider integration

The application sends chat requests to `OpenRouter` using a portfolio-specific preset configured on the OpenRouter platform and referenced by the app as the selected model. This keeps the provider integration narrow while letting prompt and assistant behavior stay centrally managed in the OpenRouter preset.

### 4. Reverse proxy

`Caddy` fronts the application in deployment. It handles:

- inbound HTTP traffic
- proxying to the app container
- coarse request handling concerns
- healthcheck-aware readiness in the deployment setup

## Primary Flows

### Public page rendering

```mermaid
sequenceDiagram
    participant B as Browser
    participant P as Proxy
    participant A as Next.js App
    participant C as Curated Content

    B->>P: Request page
    P->>A: Forward request
    A->>C: Load content
    C-->>A: Typed content data
    A-->>P: HTML response
    P-->>B: Rendered page
```

### Grounded chat request

```mermaid
sequenceDiagram
    participant B as Browser
    participant A as Chat API
    participant O as OpenRouter

    B->>A: Send prompt + short history
    A->>O: Send request through configured preset model
    O-->>A: Response
    A-->>B: Assistant answer
```

## Technology Stack

Core application stack:

- `Next.js`
- `React`
- `TypeScript`
- `Tailwind CSS`

Content:

- MDX-based curated content

AI integration:

- `OpenRouter`

Quality and validation:

- `Biome`
- `Vitest`
- `Playwright`

Deployment:

- `Docker`
- `Docker Compose`
- `Caddy`
- VPS hosting

## Deployment Topology

```mermaid
flowchart TD
    I["Internet"] --> C["Caddy Reverse Proxy"]
    C --> N["Next.js Container"]
    N --> K["OpenRouter API"]
    N --> F["Local Content Files"]
    C --> Z["/healthz checks"]
```

The deployment model favors low operational overhead:

- one application container
- one proxy container
- no data plane services
- environment-variable-based runtime configuration

## Operational Notes

This GitHub Pages repository is not the application runtime. It only redirects legacy traffic from `guilhermeledes.github.io` to the live domain.
