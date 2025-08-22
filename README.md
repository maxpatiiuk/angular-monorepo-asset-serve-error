# Angular Monorepo Asset Serve Error

In a monorepo, node_modules may be located outside the Angular project root folder.

If a library from such outside node_modules tries to load any asset, e.g. a font file from a CSS file, the import is blocked by Vite's dev server with a 403 error.

## Reproduction

1. Clone this repo

   ```sh
   git clone https://github.com/maxpatiiuk/angular-monorepo-asset-serve-error
   cd angular-monorepo-asset-serve-error
   ```

2. Install dependencies

   ```sh
   # To keep reproduction size minimal, I hardcoded a minimal root-level node_modules
   # So, install Angular dependencies only in the app folder:
   cd app
   npm install
   ```

3. Start the development server in the app folder

   ```sh
   npm start
   ```

   See this error in the browser console:

   ```
   (index):14  GET http://localhost:4200/@fs/Users/.../angular-monorepo-asset-serve-error/node_modules/external-library/NotoSans.woff2 net::ERR_ABORTED 403 (Forbidden)
   ```

## Explanation & Solution

If you run the dev server with Vite config logging (`DEBUG=vite:config npx ng serve`), you will see that the dev server was allowed to serve only the package-level node_modules folder:

```yaml
  vite:config     server: {
  vite:config       ...
  vite:config       fs: {
  vite:config         allow: [
  vite:config           '/Users/mak13180/site/esri/angular-monorepo-asset-serve-error/app/.angular/cache/20.2.0/reproduction/vite',
  vite:config           '/Users/mak13180/site/esri/angular-monorepo-asset-serve-error/app/node_modules'
  vite:config         ]
  vite:config       }
  vite:config     },
```

By default, Vite correctly detects that it is being run in a monorepo and allows serving from any monorepo node_modules, but Angular incorrectly overrides this behavior here:

https://github.com/angular/angular-cli/blob/cc05242aa8354f3280b5a70eb1f9c0e9d85f1408/packages/angular/build/src/builders/dev-server/vite/server.ts#L80-L90

The code comment states `These would be available by default but when the 'allow' option is explicitly configured, they must be included manually.`. However, Angular does not include things manually correctly.

Fortunately, the Vite's default behavior is easy to get back:

https://github.com/vitejs/vite/blob/e899bc7c73a27cdf327875e5d696c50d396a7fc2/packages/vite/src/node/server/index.ts#L1126-L1127

Their default calls `searchForWorkspaceRoot()`, which is exposed by Vite and can be called manually as documented in https://vite.dev/config/server-options.html#server-fs-allow.
