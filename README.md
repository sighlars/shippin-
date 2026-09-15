# shippin'

> An open source developer launch studio for packaging, presenting, and shipping software products.

Shippin' helps developers turn finished software into polished launch material.

Bring in a web application, API, developer tool, or supported project. Shippin' understands the product, creates structured launch content and visual assets, lets you customize the result, and exports a complete launch package.

## What Shippin' does

```text
Build your product
       ↓
Bring it into Shippin'
       ↓
Understand the product
       ↓
Create launch content and visuals
       ↓
Customize
       ↓
Export
       ↓
Ship
```

## Core capabilities

• Product inspection and Product Intelligence  
• AI assisted launch content using your own API key  
• Deterministic marketing asset generation  
• Product screenshots and presentation layouts  
• Launch cards and social graphics  
• GitHub and Product Hunt focused assets  
• Product and launch package metadata  
• Complete launch package export  
• Public product pages and discovery  
• Native desktop workflow through Wails  

## Design

Shippin' uses a distinctive dark visual identity built around:

```text
Black       #0A0A0A
Acid Lime   #B8FF3D
Off White   #F5F5F5
Dark Gray   #1A1A1A
```

The central brand mark combines a sleek ship with the letter S into a single symbol.

The visual direction is technical, editorial, premium, and developer focused.

## Architecture

Shippin' uses a monorepo containing three application targets:

```text
shippin'
│
├── Web application
├── Go backend
└── Desktop application
```

The web and desktop applications share React components, types, domain models, and application services.

The Go backend is a modular monolith. Web clients communicate with it through HTTP, while the desktop application uses Wails bindings and local capabilities.

## Technology

• Go  
• React  
• TypeScript  
• Vite  
• Wails  
• PostgreSQL  
• SQLite where local persistence is required  
• Docker  
• Playwright  
• FFmpeg  
• GitHub Actions  

## AI

AI is optional.

The open source version uses BYOK, allowing developers to connect supported AI providers with their own credentials.

Shippin' does not require a hosted Shippin' AI service for the open source MVP.

AI receives structured product context whenever possible instead of an entire repository.

Generated output is validated against structured contracts and must not invent unsupported product claims.

## Visual generation

Shippin' does not rely on AI to control exact visual layouts.

The core rendering approach is:

```text
Product data
     ↓
Template
     ↓
HTML / CSS / SVG
     ↓
Browser rendering
     ↓
PNG / JPEG
```

This keeps generated assets deterministic, editable, and reproducible.

## Security principles

Imported projects are treated as untrusted.

The open source web application does not execute arbitrary user repositories on the server.

Local project execution, where supported, is isolated through the desktop application's local sandbox capabilities.

API keys must never be logged.

Server side URL inspection must protect against SSRF.

Generated content and rendered assets must be validated before use.

## Project status

Shippin' is being built as an open source MVP.

The initial focus is the core packaging workflow:

```text
Inspect
   ↓
Understand
   ↓
Generate
   ↓
Customize
   ↓
Render
   ↓
Export
```

Public discovery is included as a distribution surface, but it is not the primary focus of the landing experience.

## Development

The project is designed to support a zero cost local development setup using open source tooling and free development resources.

See the project specification for the complete architecture, implementation phases, design system, asset rules, and development requirements.

## Contributing

Contributions are welcome.

Before making architectural changes, read the project specification and existing documentation. Keep the modular monolith architecture intact unless there is a concrete reason to change it.

Avoid adding infrastructure or dependencies without a clear requirement.

## License

License information will be added with the project release.
