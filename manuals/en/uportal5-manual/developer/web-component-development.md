# Web Component Development

These should be viewed as a set of 'best practice' guidelines or recommendations when developing web components geared toward the uPortal ecosystem. While web components can be built in a variety of ways, the goal is a simple and consistent approach in development, style, and deployment.

## Build Pipeline
Until the frontend is more separated from the backend, the webjar > maven > resource-server is the approach uPortal takes to host the web components for usage on the site.

LIT & Typescript (Transpiled to ES6+)
 v
Linters
 v
Pre-commit hooks
 v
Deployed to Sonatype
 v
Bundled as a webjar
 v
Deployed to Maven
 v
Pulled into uPortal projects via the resource-server
 v
Pull into an HTML page via a link from the resource-server

## Technology
To keep this document concise, please refer to the reference implementation for details / examples of the various configurations.

### NPM
https://www.npmjs.com/

The build is handled via NPM.

### Lerna
https://lerna.js.org/
https://github.com/ChristianMurphy/lerna-webjar

Builds the webjar

### Lit 
https://lit.dev/

Lightweight method of creating Web Components.

### Typescript
https://www.typescriptlang.org/

Type checking.

### ESLint
https://eslint.org/

Linter for Javascript.

### StyleLint
https://stylelint.io/

Linter for CSS.

### Remark
https://github.com/remarkjs/remark-lint

Linter for markdown.

### Prettier
https://prettier.io/

Consistent code format for MD, HTML, CSS, etc.

### Editor Config

Consistent code style in the IDE.

## Reference Implementation

The following web component aims to follow these best practices:

```
TBD - build out a web component that follows these guidelines and interacts with uPortal (ideally with CRUD operations for example's sake)
```