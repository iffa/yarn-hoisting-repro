# yarn-hoisting-repro

## Background info

```yml
# use nm linker
nodeLinker: node-modules

# hoisting limits default to none
nmHoistingLimits: none
```

mobile has `installConfig.hoistingLimits` set to `workspaces` in `package.json` to contain its dependencies (and make RN work)

## Reproduce issue

`corepack enable`

`yarn install`

-> web-common workspace shouldn't have `react-query` in its own node_modules

## What is happening

### yarn why

```text
➜  yarn-hoisting-repro git:(main) ✗ yarn why @tanstack/react-query
├─ mobile@workspace:mobile
│  └─ @tanstack/react-query@npm:4.36.1 [3eef6] (via npm:^4.36.1 [3eef6])
│
├─ web-common@workspace:web-common
│  └─ @tanstack/react-query@npm:5.55.4 [b942b] (via npm:^5.55.4 [b942b])
│
└─ web@workspace:web
   └─ @tanstack/react-query@npm:5.55.4 [b942b] (via npm:^5.55.4 [b942b])
```

### yarn info

```text
➜  yarn-hoisting-repro git:(main) ✗ yarn info -AR --virtuals --dependents @tanstack/react-query

├─ @tanstack/react-query@npm:4.36.1
│  ├─ Instances: 1
│  ├─ Version: 4.36.1
│  │
│  └─ Dependencies
│     ├─ @tanstack/query-core@npm:4.36.1 → npm:4.36.1
│     └─ use-sync-external-store@npm:^1.2.0 → npm:1.2.2
│
├─ @tanstack/react-query@npm:5.55.4
│  ├─ Instances: 1
│  ├─ Version: 5.55.4
│  │
│  └─ Dependencies
│     └─ @tanstack/query-core@npm:5.55.4 → npm:5.55.4
│
├─ @tanstack/react-query@npm:4.36.1 [3eef6]
│  ├─ Version: 4.36.1
│  │
│  ├─ Dependents
│  │  └─ mobile@workspace:mobile
│  │
│  └─ Peer dependencies
│     ├─ @types/react-dom@* → ✘
│     ├─ @types/react-native@* → ✘
│     ├─ @types/react@* → ✘
│     ├─ react-dom@^16.8.0 || ^17.0.0 || ^18.0.0 → ✘
│     ├─ react-native@* → npm:0.73.9 [3eef6]
│     └─ react@^16.8.0 || ^17.0.0 || ^18.0.0 → npm:18.2.0
│
└─ @tanstack/react-query@npm:5.55.4 [b942b]
   ├─ Version: 5.55.4
   │
   ├─ Dependents
   │  ├─ web-common@workspace:web-common
   │  └─ web@workspace:web
   │
   └─ Peer dependencies
      ├─ @types/react@* → ✘
      └─ react@^18 || ^19 → npm:18.3.1
```

### yarn install with NM_DEBUG_LEVEL=2

```text
➜  yarn-hoisting-repro git:(main) ✗ NM_DEBUG_LEVEL=2 yarn install
➤ YN0000: · Yarn 4.4.1
➤ YN0000: ┌ Resolution step
➤ YN0000: └ Completed
➤ YN0000: ┌ Post-resolution validation
➤ YN0086: │ Some peer dependencies are incorrectly met by dependencies; run yarn explain peer-requirements for details.
➤ YN0000: └ Completed
➤ YN0000: ┌ Fetch step
➤ YN0000: └ Completed in 0s 205ms
➤ YN0000: ┌ Link step
hoist time: 23ms, rounds: 1
➤ YN0001: │ Error: The hoisting result is not terminal, prev tree:
├─@tanstack/query-core@5.55.4
├─js-tokens@4.0.0
├─loose-envify@1.4.0
├─w:wh:mobile
│ ├─@ampproject/remapping@2.3.0
│ ├─@babel/code-frame@7.24.7
│ ├─@babel/compat-data@7.25.4
│ ├─@babel/core@7.25.2
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/generator@7.25.6
│ ├─@babel/helper-annotate-as-pure@7.24.7
│ ├─@babel/helper-compilation-targets@7.25.2
│ │ ├─lru-cache@5.1.1 - filled by: lru-cache@10.4.3 at .→w:wh:mobile
│ │ └─yallist@3.1.1
│ ├─v:@babel/helper-create-class-features-plugin@7.25.4
│ ├─v:@babel/helper-create-regexp-features-plugin@7.25.2
│ ├─v:@babel/helper-define-polyfill-provider@0.6.2
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/helper-environment-visitor@7.24.7
│ ├─@babel/helper-member-expression-to-functions@7.24.8
│ ├─@babel/helper-module-imports@7.24.7
│ ├─v:@babel/helper-module-transforms@7.25.2
│ ├─@babel/helper-optimise-call-expression@7.24.7
│ ├─@babel/helper-plugin-utils@7.24.8
│ ├─v:@babel/helper-remap-async-to-generator@7.25.0
│ ├─v:@babel/helper-replace-supers@7.25.0
│ ├─@babel/helper-simple-access@7.24.7
│ ├─@babel/helper-skip-transparent-expression-wrappers@7.24.7
│ ├─@babel/helper-string-parser@7.24.8
│ ├─@babel/helper-validator-identifier@7.24.7
│ ├─@babel/helper-validator-option@7.24.8
│ ├─@babel/helper-wrap-function@7.25.0
│ ├─@babel/helpers@7.25.6
│ ├─@babel/highlight@7.24.7
│ │ ├─ansi-styles@3.2.1
│ │ ├─chalk@2.4.2 - filled by: chalk@4.1.2 at .→w:wh:mobile
│ │ ├─color-convert@1.9.3
│ │ ├─color-name@1.1.3
│ │ ├─escape-string-regexp@1.0.5
│ │ ├─has-flag@3.0.0
│ │ └─supports-color@5.5.0
│ ├─@babel/parser@7.25.6
│ ├─v:@babel/plugin-proposal-async-generator-functions@7.20.7
│ ├─v:@babel/plugin-proposal-class-properties@7.18.6
│ ├─v:@babel/plugin-proposal-export-default-from@7.24.7
│ ├─v:@babel/plugin-proposal-nullish-coalescing-operator@7.18.6
│ ├─v:@babel/plugin-proposal-numeric-separator@7.18.6
│ ├─v:@babel/plugin-proposal-object-rest-spread@7.20.7
│ ├─v:@babel/plugin-proposal-optional-catch-binding@7.18.6
│ ├─v:@babel/plugin-proposal-optional-chaining@7.21.0
│ ├─v:@babel/plugin-syntax-async-generators@7.8.4
│ ├─v:@babel/plugin-syntax-dynamic-import@7.8.3
│ ├─v:@babel/plugin-syntax-export-default-from@7.24.7
│ ├─v:@babel/plugin-syntax-flow@7.24.7
│ ├─v:@babel/plugin-syntax-jsx@7.24.7
│ ├─v:@babel/plugin-syntax-nullish-coalescing-operator@7.8.3
│ ├─v:@babel/plugin-syntax-numeric-separator@7.10.4
│ ├─v:@babel/plugin-syntax-object-rest-spread@7.8.3
│ ├─v:@babel/plugin-syntax-optional-catch-binding@7.8.3
│ ├─v:@babel/plugin-syntax-optional-chaining@7.8.3
│ ├─v:@babel/plugin-syntax-private-property-in-object@7.14.5
│ ├─v:@babel/plugin-syntax-typescript@7.25.4
│ ├─v:@babel/plugin-transform-arrow-functions@7.24.7
│ ├─v:@babel/plugin-transform-async-to-generator@7.24.7
│ ├─v:@babel/plugin-transform-block-scoping@7.25.0
│ ├─v:@babel/plugin-transform-classes@7.25.4
│ ├─v:@babel/plugin-transform-computed-properties@7.24.7
│ ├─v:@babel/plugin-transform-destructuring@7.24.8
│ ├─v:@babel/plugin-transform-flow-strip-types@7.25.2
│ ├─v:@babel/plugin-transform-function-name@7.25.1
│ ├─v:@babel/plugin-transform-literals@7.25.2
│ ├─v:@babel/plugin-transform-modules-commonjs@7.24.8
│ ├─v:@babel/plugin-transform-named-capturing-groups-regex@7.24.7
│ ├─v:@babel/plugin-transform-parameters@7.24.7
│ ├─v:@babel/plugin-transform-private-methods@7.25.4
│ ├─v:@babel/plugin-transform-private-property-in-object@7.24.7
│ ├─v:@babel/plugin-transform-react-display-name@7.24.7
│ ├─v:@babel/plugin-transform-react-jsx@7.25.2
│ ├─v:@babel/plugin-transform-react-jsx-self@7.24.7
│ ├─v:@babel/plugin-transform-react-jsx-source@7.24.7
│ ├─v:@babel/plugin-transform-runtime@7.25.4
│ ├─v:@babel/plugin-transform-shorthand-properties@7.24.7
│ ├─v:@babel/plugin-transform-spread@7.24.7
│ ├─v:@babel/plugin-transform-sticky-regex@7.24.7
│ ├─v:@babel/plugin-transform-typescript@7.25.2
│ ├─v:@babel/plugin-transform-unicode-regex@7.24.7
│ ├─v:@babel/preset-flow@7.24.7
│ ├─v:@babel/preset-typescript@7.24.7
│ ├─v:@babel/register@7.24.6
│ ├─@babel/regjsgen@0.8.0
│ ├─@babel/runtime@7.25.6
│ ├─@babel/template@7.25.0
│ ├─@babel/traverse@7.25.6
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/types@7.25.6
│ ├─@hapi/hoek@9.3.0
│ ├─@hapi/topo@5.1.0
│ ├─@isaacs/cliui@8.0.2
│ │ ├─ansi-regex@6.1.0
│ │ ├─ansi-styles@6.2.1
│ │ ├─emoji-regex@9.2.2
│ │ ├─string-width@5.1.2 - filled by: string-width@4.2.3 at .→w:wh:mobile
│ │ ├─strip-ansi@7.1.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ │ └─wrap-ansi@8.1.0 - filled by: wrap-ansi@6.2.0 at .→w:wh:mobile
│ ├─@isaacs/ttlcache@1.4.1
│ ├─@jest/create-cache-key-function@29.7.0
│ ├─@jest/environment@29.7.0
│ ├─@jest/fake-timers@29.7.0
│ ├─@jest/schemas@29.6.3
│ ├─@jest/types@29.6.3
│ ├─@jridgewell/gen-mapping@0.3.5
│ ├─@jridgewell/resolve-uri@3.1.2
│ ├─@jridgewell/set-array@1.2.1
│ ├─@jridgewell/source-map@0.3.6
│ ├─@jridgewell/sourcemap-codec@1.5.0
│ ├─@jridgewell/trace-mapping@0.3.25
│ ├─@npmcli/agent@2.2.2
│ ├─@npmcli/fs@3.1.1
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@pkgjs/parseargs@0.11.0
│ ├─@react-native-community/cli@12.3.7
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-clean@12.3.7
│ ├─@react-native-community/cli-config@12.3.7
│ ├─@react-native-community/cli-debugger-ui@12.3.7
│ ├─@react-native-community/cli-doctor@12.3.7
│ │ ├─ansi-regex@4.1.1
│ │ ├─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ │ └─strip-ansi@5.2.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-hermes@12.3.7
│ ├─@react-native-community/cli-platform-android@12.3.7
│ ├─@react-native-community/cli-platform-ios@12.3.7
│ ├─@react-native-community/cli-plugin-metro@12.3.7
│ ├─@react-native-community/cli-server-api@12.3.7
│ ├─@react-native-community/cli-tools@12.3.7
│ │ ├─find-up@5.0.0 - filled by: find-up@4.1.0 at .→w:wh:mobile
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-types@12.3.7
│ ├─@react-native/assets-registry@0.73.1
│ ├─@react-native/babel-plugin-codegen@0.73.4
│ ├─v:@react-native/babel-preset@0.73.21
│ ├─v:@react-native/codegen@0.73.3
│ │ └─mkdirp@0.5.6 - filled by: mkdirp@1.0.4 at .→w:wh:mobile
│ ├─@react-native/community-cli-plugin@0.73.18
│ ├─@react-native/debugger-frontend@0.73.3
│ ├─@react-native/dev-middleware@0.73.8
│ │ ├─open@7.4.2 - filled by: open@6.4.0 at .→w:wh:mobile
│ │ └─v:ws@6.2.3 - filled by: ws@7.5.10 at .→w:wh:mobile
│ ├─@react-native/gradle-plugin@0.73.4
│ ├─@react-native/js-polyfills@0.73.1
│ ├─v:@react-native/metro-babel-transformer@0.73.15
│ │ ├─hermes-estree@0.15.0
│ │ └─hermes-parser@0.15.0 - filled by: hermes-parser@0.23.1 at .→w:wh:mobile
│ ├─@react-native/normalize-colors@0.73.2
│ ├─v:@react-native/virtualized-lists@0.73.4
│ ├─@sideway/address@4.1.5
│ ├─@sideway/formula@3.0.1
│ ├─@sideway/pinpoint@2.0.0
│ ├─@sinclair/typebox@0.27.8
│ ├─@sinonjs/commons@3.0.1
│ ├─@sinonjs/fake-timers@10.3.0
│ ├─@tanstack/query-core@4.36.1
│ ├─v:@tanstack/react-query@4.36.1
│ ├─@types/istanbul-lib-coverage@2.0.6
│ ├─@types/istanbul-lib-report@3.0.3
│ ├─@types/istanbul-reports@3.0.4
│ ├─@types/node@22.5.4
│ ├─@types/stack-utils@2.0.3
│ ├─@types/yargs@17.0.33
│ ├─@types/yargs-parser@21.0.3
│ ├─abbrev@2.0.0
│ ├─abort-controller@3.0.0
│ ├─accepts@1.3.8
│ ├─acorn@8.12.1
│ ├─agent-base@7.1.1
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─aggregate-error@3.1.0
│ ├─anser@1.4.10
│ ├─ansi-fragments@0.2.1
│ │ ├─ansi-regex@4.1.1
│ │ └─strip-ansi@5.2.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ ├─ansi-regex@5.0.1
│ ├─ansi-styles@4.3.0
│ ├─anymatch@3.1.3
│ ├─appdirsjs@1.2.7
│ ├─argparse@1.0.10
│ ├─asap@2.0.6
│ ├─ast-types@0.15.2
│ ├─astral-regex@1.0.0
│ ├─async-limiter@1.0.1
│ ├─v:babel-core@7.0.0-bridge.0
│ ├─v:babel-plugin-polyfill-corejs2@0.4.11
│ ├─v:babel-plugin-polyfill-corejs3@0.10.6
│ ├─v:babel-plugin-polyfill-regenerator@0.6.2
│ ├─babel-plugin-transform-flow-enums@0.0.2
│ ├─balanced-match@1.0.2
│ ├─base64-js@1.5.1
│ ├─bl@4.1.0
│ ├─brace-expansion@1.1.11
│ ├─braces@3.0.3
│ ├─browserslist@4.23.3
│ ├─bser@2.1.1
│ ├─buffer@5.7.1
│ ├─buffer-from@1.1.2
│ ├─bytes@3.0.0
│ ├─cacache@18.0.4
│ │ ├─brace-expansion@2.0.1
│ │ ├─glob@10.4.5 - filled by: glob@7.2.3 at .→w:wh:mobile
│ │ └─minimatch@9.0.5
│ ├─caller-callsite@2.0.0
│ ├─caller-path@2.0.0
│ ├─callsites@2.0.0
│ ├─camelcase@5.3.1
│ ├─caniuse-lite@1.0.30001660
│ ├─chalk@4.1.2
│ ├─chownr@2.0.0
│ ├─chrome-launcher@0.15.2
│ ├─chromium-edge-launcher@1.0.0
│ ├─ci-info@2.0.0
│ ├─clean-stack@2.2.0
│ ├─cli-cursor@3.1.0
│ ├─cli-spinners@2.9.2
│ ├─cliui@6.0.0
│ ├─clone@1.0.4
│ ├─clone-deep@4.0.1
│ ├─color-convert@2.0.1
│ ├─color-name@1.1.4
│ ├─colorette@1.4.0
│ ├─command-exists@1.2.9
│ ├─commander@9.5.0
│ ├─commondir@1.0.1
│ ├─compressible@2.0.18
│ │ └─mime-db@1.53.0 - filled by: mime-db@1.52.0 at .→w:wh:mobile
│ ├─compression@1.7.4
│ ├─concat-map@0.0.1
│ ├─connect@3.7.0
│ ├─convert-source-map@2.0.0
│ ├─core-js-compat@3.38.1
│ ├─core-util-is@1.0.3
│ ├─cosmiconfig@5.2.1
│ ├─cross-spawn@7.0.3
│ ├─dayjs@1.11.13
│ ├─v:debug@2.6.9
│ │ └─ms@2.0.0 - filled by: ms@2.1.3 at .→w:wh:mobile
│ ├─decamelize@1.2.0
│ ├─deepmerge@4.3.1
│ ├─defaults@1.0.4
│ ├─denodeify@1.2.1
│ ├─depd@2.0.0
│ ├─deprecated-react-native-prop-types@5.0.0
│ ├─destroy@1.2.0
│ ├─eastasianwidth@0.2.0
│ ├─ee-first@1.1.1
│ ├─electron-to-chromium@1.5.18
│ ├─emoji-regex@8.0.0
│ ├─encodeurl@1.0.2
│ ├─encoding@0.1.13
│ ├─env-paths@2.2.1
│ ├─envinfo@7.13.0
│ ├─err-code@2.0.3
│ ├─error-ex@1.3.2
│ ├─error-stack-parser@2.1.4
│ ├─errorhandler@1.5.1
│ ├─escalade@3.2.0
│ ├─escape-html@1.0.3
│ ├─escape-string-regexp@4.0.0
│ ├─esprima@4.0.1
│ ├─etag@1.8.1
│ ├─event-target-shim@5.0.1
│ ├─execa@5.1.1
│ ├─exponential-backoff@3.1.1
│ ├─fast-xml-parser@4.5.0
│ ├─fb-watchman@2.0.2
│ ├─fill-range@7.1.1
│ ├─finalhandler@1.1.2
│ │ ├─on-finished@2.3.0 - filled by: on-finished@2.4.1 at .→w:wh:mobile
│ │ └─statuses@1.5.0 - filled by: statuses@2.0.1 at .→w:wh:mobile
│ ├─find-cache-dir@2.1.0
│ ├─find-up@4.1.0
│ │ ├─locate-path@5.0.0 - filled by: locate-path@6.0.0 at .→w:wh:mobile
│ │ └─p-locate@4.1.0
│ ├─flow-enums-runtime@0.0.6
│ ├─flow-parser@0.206.0
│ ├─foreground-child@3.3.0
│ │ └─signal-exit@4.1.0 - filled by: signal-exit@3.0.7 at .→w:wh:mobile
│ ├─fresh@0.5.2
│ ├─fs-extra@8.1.0
│ ├─fs-minipass@3.0.3
│ ├─fs.realpath@1.0.0
│ ├─fsevents@optional!builtin<compat/fsevents>::version=2.3.3&hash=df0bf1
│ ├─function-bind@1.1.2
│ ├─gensync@1.0.0-beta.2
│ ├─get-caller-file@2.0.5
│ ├─get-stream@6.0.1
│ ├─glob@7.2.3
│ ├─globals@11.12.0
│ ├─graceful-fs@4.2.11
│ ├─has-flag@4.0.0
│ ├─hasown@2.0.2
│ ├─hermes-estree@0.23.1
│ ├─hermes-parser@0.23.1
│ ├─hermes-profile-transformer@0.0.6
│ │ └─source-map@0.7.4 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─http-cache-semantics@4.1.1
│ ├─http-errors@2.0.0
│ ├─http-proxy-agent@7.0.2
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─https-proxy-agent@7.0.5
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─human-signals@2.1.0
│ ├─iconv-lite@0.6.3
│ ├─ieee754@1.2.1
│ ├─image-size@1.1.1
│ ├─import-fresh@2.0.0
│ ├─imurmurhash@0.1.4
│ ├─indent-string@4.0.0
│ ├─inflight@1.0.6
│ ├─inherits@2.0.4
│ ├─invariant@2.2.4
│ ├─ip-address@9.0.5
│ │ └─sprintf-js@1.1.3 - filled by: sprintf-js@1.0.3 at .→w:wh:mobile
│ ├─is-arrayish@0.2.1
│ ├─is-core-module@2.15.1
│ ├─is-directory@0.3.1
│ ├─is-docker@2.2.1
│ ├─is-fullwidth-code-point@2.0.0
│ ├─is-interactive@1.0.0
│ ├─is-lambda@1.0.1
│ ├─is-number@7.0.0
│ ├─is-plain-object@2.0.4
│ ├─is-stream@2.0.1
│ ├─is-unicode-supported@0.1.0
│ ├─is-wsl@2.2.0
│ ├─isarray@1.0.0
│ ├─isexe@2.0.0
│ ├─isobject@3.0.1
│ ├─jackspeak@3.4.3
│ ├─jest-environment-node@29.7.0
│ ├─jest-get-type@29.6.3
│ ├─jest-message-util@29.7.0
│ │ ├─ansi-styles@5.2.0
│ │ └─pretty-format@29.7.0 - filled by: pretty-format@26.6.2 at .→w:wh:mobile
│ ├─jest-mock@29.7.0
│ ├─jest-util@29.7.0
│ │ ├─ci-info@3.9.0 - filled by: ci-info@2.0.0 at .→w:wh:mobile
│ ├─jest-validate@29.7.0
│ │ ├─ansi-styles@5.2.0
│ │ ├─camelcase@6.3.0 - filled by: camelcase@5.3.1 at .→w:wh:mobile
│ │ └─pretty-format@29.7.0 - filled by: pretty-format@26.6.2 at .→w:wh:mobile
│ ├─jest-worker@29.7.0
│ │ └─supports-color@8.1.1 - filled by: supports-color@7.2.0 at .→w:wh:mobile
│ ├─joi@17.13.3
│ ├─js-tokens@4.0.0
│ ├─js-yaml@3.14.1
│ ├─jsbn@1.1.0
│ ├─jsc-android@250231.0.0
│ ├─jsc-safe-url@0.2.4
│ ├─v:jscodeshift@0.14.0
│ │ ├─flow-parser@0.245.2 - filled by: flow-parser@0.206.0 at .→w:wh:mobile
│ ├─jsesc@2.5.2
│ ├─json-parse-better-errors@1.0.2
│ ├─json5@2.2.3
│ ├─jsonfile@4.0.0
│ ├─kind-of@6.0.3
│ ├─kleur@3.0.3
│ ├─leven@3.1.0
│ ├─lighthouse-logger@1.4.2
│ ├─locate-path@6.0.0
│ ├─lodash.debounce@4.0.8
│ ├─lodash.throttle@4.1.1
│ ├─log-symbols@4.1.0
│ ├─logkitty@0.7.1
│ │ └─yargs@15.4.1 - filled by: yargs@17.7.2 at .→w:wh:mobile
│ ├─loose-envify@1.4.0
│ ├─lru-cache@10.4.3
│ ├─make-dir@2.1.0
│ │ └─semver@5.7.2 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─make-fetch-happen@13.0.1
│ ├─makeerror@1.0.12
│ ├─marky@1.2.5
│ ├─memoize-one@5.2.1
│ ├─merge-stream@2.0.0
│ ├─metro@0.80.12
│ ├─metro-babel-transformer@0.80.12
│ ├─metro-cache@0.80.12
│ ├─metro-cache-key@0.80.12
│ ├─metro-config@0.80.12
│ ├─metro-core@0.80.12
│ ├─metro-file-map@0.80.12
│ ├─metro-minify-terser@0.80.12
│ ├─metro-resolver@0.80.12
│ ├─metro-runtime@0.80.12
│ ├─metro-source-map@0.80.12
│ ├─metro-symbolicate@0.80.12
│ ├─metro-transform-plugins@0.80.12
│ ├─metro-transform-worker@0.80.12
│ ├─micromatch@4.0.8
│ ├─mime@2.6.0
│ ├─mime-db@1.52.0
│ ├─mime-types@2.1.35
│ ├─mimic-fn@2.1.0
│ ├─minimatch@3.1.2
│ ├─minimist@1.2.8
│ ├─minipass@7.1.2
│ ├─minipass-collect@2.0.1
│ ├─minipass-fetch@3.0.5
│ ├─minipass-flush@1.0.5
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minipass-pipeline@1.2.4
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minipass-sized@1.0.3
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minizlib@2.1.2
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─mkdirp@1.0.4
│ ├─w:mobile
│ ├─ms@2.1.3
│ ├─negotiator@0.6.3
│ ├─neo-async@2.6.2
│ ├─nocache@3.0.4
│ ├─node-abort-controller@3.1.1
│ ├─node-dir@0.1.17
│ ├─v:node-fetch@2.7.0
│ ├─node-gyp@10.2.0
│ │ ├─brace-expansion@2.0.1
│ │ ├─glob@10.4.5 - filled by: glob@7.2.3 at .→w:wh:mobile
│ │ ├─isexe@3.1.1
│ │ ├─minimatch@9.0.5
│ │ ├─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ │ └─which@4.0.0 - filled by: which@2.0.2 at .→w:wh:mobile
│ ├─node-int64@0.4.0
│ ├─node-releases@2.0.18
│ ├─node-stream-zip@1.15.0
│ ├─nopt@7.2.1
│ ├─normalize-path@3.0.0
│ ├─npm-run-path@4.0.1
│ ├─nullthrows@1.1.1
│ ├─ob1@0.80.12
│ ├─object-assign@4.1.1
│ ├─on-finished@2.4.1
│ ├─on-headers@1.0.2
│ ├─once@1.4.0
│ ├─onetime@5.1.2
│ ├─open@6.4.0
│ │ ├─is-wsl@1.1.0 - filled by: is-wsl@2.2.0 at .→w:wh:mobile
│ ├─ora@5.4.1
│ ├─p-limit@2.3.0
│ ├─p-locate@5.0.0
│ │ ├─p-limit@3.1.0 - filled by: p-limit@2.3.0 at .→w:wh:mobile
│ ├─p-map@4.0.0
│ ├─p-try@2.2.0
│ ├─package-json-from-dist@1.0.0
│ ├─parse-json@4.0.0
│ ├─parseurl@1.3.3
│ ├─path-exists@4.0.0
│ ├─path-is-absolute@1.0.1
│ ├─path-key@3.1.1
│ ├─path-parse@1.0.7
│ ├─path-scurry@1.11.1
│ ├─picocolors@1.1.0
│ ├─picomatch@2.3.1
│ ├─pify@4.0.1
│ ├─pirates@4.0.6
│ ├─pkg-dir@3.0.0
│ │ ├─find-up@3.0.0 - filled by: find-up@4.1.0 at .→w:wh:mobile
│ │ ├─locate-path@3.0.0
│ │ ├─p-locate@3.0.0
│ │ ├─path-exists@3.0.0
│ ├─pretty-format@26.6.2
│ │ ├─@jest/types@26.6.2 - filled by: @jest/types@29.6.3 at .→w:wh:mobile
│ │ ├─@types/yargs@15.0.19
│ │ └─react-is@17.0.2 - filled by: react-is@18.3.1 at .→w:wh:mobile
│ ├─proc-log@4.2.0
│ ├─process-nextick-args@2.0.1
│ ├─promise@8.3.0
│ ├─promise-retry@2.0.1
│ ├─prompts@2.4.2
│ ├─prop-types@15.8.1
│ │ └─react-is@16.13.1 - filled by: react-is@18.3.1 at .→w:wh:mobile
│ ├─queue@6.0.2
│ ├─range-parser@1.2.1
│ ├─react@18.2.0
│ ├─react-devtools-core@4.28.5
│ ├─react-is@18.3.1
│ ├─v:react-native@0.73.9
│ │ ├─mkdirp@0.5.6 - filled by: mkdirp@1.0.4 at .→w:wh:mobile
│ │ ├─regenerator-runtime@0.13.11 - filled by: regenerator-runtime@0.14.1 at .→w:wh:mobile
│ │ └─v:ws@6.2.3 - filled by: ws@7.5.10 at .→w:wh:mobile
│ ├─react-refresh@0.14.2
│ ├─v:react-shallow-renderer@16.15.0
│ ├─readable-stream@3.6.2
│ ├─readline@1.3.0
│ ├─recast@0.21.5
│ │ └─source-map@0.6.1 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─regenerate@1.4.2
│ ├─regenerate-unicode-properties@10.1.1
│ ├─regenerator-runtime@0.14.1
│ ├─regexpu-core@5.3.2
│ ├─regjsparser@0.9.1
│ │ ├─jsesc@0.5.0 - filled by: jsesc@2.5.2 at .→w:wh:mobile
│ ├─require-directory@2.1.1
│ ├─require-main-filename@2.0.0
│ ├─resolve@optional!builtin<compat/resolve>::version=1.22.8&hash=c3c19d
│ ├─resolve-from@3.0.0
│ ├─restore-cursor@3.1.0
│ ├─retry@0.12.0
│ ├─rimraf@3.0.2
│ ├─safe-buffer@5.1.2
│ ├─safer-buffer@2.1.2
│ ├─scheduler@0.24.0-canary-efb381bbf-20230505
│ ├─semver@6.3.1
│ ├─send@0.18.0
│ │ ├─mime@1.6.0 - filled by: mime@2.6.0 at .→w:wh:mobile
│ ├─serialize-error@2.1.0
│ ├─serve-static@1.16.0
│ ├─set-blocking@2.0.0
│ ├─setprototypeof@1.2.0
│ ├─shallow-clone@3.0.1
│ ├─shebang-command@2.0.0
│ ├─shebang-regex@3.0.0
│ ├─shell-quote@1.8.1
│ ├─signal-exit@3.0.7
│ ├─sisteransi@1.0.5
│ ├─slash@3.0.0
│ ├─slice-ansi@2.1.0
│ │ ├─ansi-styles@3.2.1 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ ├─color-convert@1.9.3
│ │ ├─color-name@1.1.3
│ ├─smart-buffer@4.2.0
│ ├─socks@2.8.3
│ ├─socks-proxy-agent@8.0.4
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─source-map@0.5.7
│ ├─source-map-support@0.5.21
│ │ ├─source-map@0.6.1 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─sprintf-js@1.0.3
│ ├─ssri@10.0.6
│ ├─stack-utils@2.0.6
│ │ ├─escape-string-regexp@2.0.0 - filled by: escape-string-regexp@4.0.0 at .→w:wh:mobile
│ ├─stackframe@1.3.4
│ ├─stacktrace-parser@0.1.10
│ ├─statuses@2.0.1
│ ├─string-width@4.2.3
│ │ ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ ├─a:string-width-cjs:string-width@4.2.3
│ │ ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ │ └─string-width@4.2.3 - self-reference
│ │   ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ ├─string_decoder@1.3.0
│ │ ├─safe-buffer@5.2.1 - filled by: safe-buffer@5.1.2 at .→w:wh:mobile
│ ├─strip-ansi@6.0.1
│ ├─a:strip-ansi-cjs:strip-ansi@6.0.1
│ │ └─strip-ansi@6.0.1 - self-reference
│ ├─strip-final-newline@2.0.0
│ ├─strnum@1.0.5
│ ├─sudo-prompt@9.2.1
│ ├─supports-color@7.2.0
│ ├─supports-preserve-symlinks-flag@1.0.0
│ ├─tar@6.2.1
│ │ ├─fs-minipass@2.1.0 - filled by: fs-minipass@3.0.3 at .→w:wh:mobile
│ │ │ └─minipass@3.3.6 - filled by minipass@5.0.0 at w:wh:mobile
│ │ ├─minipass@5.0.0 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─temp@0.8.4
│ │ ├─rimraf@2.6.3 - filled by: rimraf@3.0.2 at .→w:wh:mobile
│ ├─temp-dir@2.0.0
│ ├─terser@5.32.0
│ │ ├─commander@2.20.3 - filled by: commander@9.5.0 at .→w:wh:mobile
│ ├─throat@5.0.0
│ ├─through2@2.0.5
│ │ ├─readable-stream@2.3.8 - filled by: readable-stream@3.6.2 at .→w:wh:mobile
│ │ ├─string_decoder@1.1.1
│ ├─tmpl@1.0.5
│ ├─to-fast-properties@2.0.0
│ ├─to-regex-range@5.0.1
│ ├─toidentifier@1.0.1
│ ├─tr46@0.0.3
│ ├─tslib@2.7.0
│ ├─type-detect@4.0.8
│ ├─type-fest@0.7.1
│ ├─undici-types@6.19.8
│ ├─unicode-canonical-property-names-ecmascript@2.0.0
│ ├─unicode-match-property-ecmascript@2.0.0
│ ├─unicode-match-property-value-ecmascript@2.1.0
│ ├─unicode-property-aliases-ecmascript@2.1.0
│ ├─unique-filename@3.0.0
│ ├─unique-slug@4.0.0
│ ├─universalify@0.1.2
│ ├─unpipe@1.0.0
│ ├─v:update-browserslist-db@1.1.0
│ ├─v:use-sync-external-store@1.2.2
│ ├─util-deprecate@1.0.2
│ ├─utils-merge@1.0.1
│ ├─vary@1.1.2
│ ├─vlq@1.0.1
│ ├─walker@1.0.8
│ ├─wcwidth@1.0.1
│ ├─webidl-conversions@3.0.1
│ ├─whatwg-fetch@3.6.20
│ ├─whatwg-url@5.0.0
│ ├─which@2.0.2
│ ├─which-module@2.0.1
│ ├─wrap-ansi@6.2.0
│ ├─a:wrap-ansi-cjs:wrap-ansi@7.0.0
│ │ └─wrap-ansi@7.0.0 - self-reference
│ ├─wrappy@1.0.2
│ ├─write-file-atomic@2.4.3
│ ├─v:ws@7.5.10
│ ├─xtend@4.0.2
│ ├─y18n@4.0.3
│ ├─yallist@4.0.0
│ ├─yaml@2.5.1
│ ├─yargs@17.7.2
│ │ ├─cliui@8.0.1 - filled by: cliui@6.0.0 at .→w:wh:mobile
│ │ ├─wrap-ansi@7.0.0
│ │ ├─y18n@5.0.8 - filled by: y18n@4.0.3 at .→w:wh:mobile
│ │ └─yargs-parser@21.1.1 - filled by: yargs-parser@18.1.3 at .→w:wh:mobile
│ ├─yargs-parser@18.1.3
│ └─yocto-queue@0.1.0
├─react@18.3.1
├─w:web
├─w:wh:web
│ └─v:@tanstack/react-query@5.55.4 - peer dependency react@18.3.1 from parent w:wh:web was not hoisted
├─w:web-common
├─w:wh:web-common
│ └─v:@tanstack/react-query@5.55.4 - peer dependency react@18.3.1 from parent w:wh:web-common was not hoisted
, next tree:
├─@tanstack/query-core@5.55.4
├─v:@tanstack/react-query@5.55.4
├─js-tokens@4.0.0
├─loose-envify@1.4.0
├─w:wh:mobile
│ ├─@ampproject/remapping@2.3.0
│ ├─@babel/code-frame@7.24.7
│ ├─@babel/compat-data@7.25.4
│ ├─@babel/core@7.25.2
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/generator@7.25.6
│ ├─@babel/helper-annotate-as-pure@7.24.7
│ ├─@babel/helper-compilation-targets@7.25.2
│ │ ├─lru-cache@5.1.1 - filled by: lru-cache@10.4.3 at .→w:wh:mobile
│ │ └─yallist@3.1.1 - filled by: yallist@4.0.0 at .→w:wh:mobile
│ ├─v:@babel/helper-create-class-features-plugin@7.25.4
│ ├─v:@babel/helper-create-regexp-features-plugin@7.25.2
│ ├─v:@babel/helper-define-polyfill-provider@0.6.2
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/helper-environment-visitor@7.24.7
│ ├─@babel/helper-member-expression-to-functions@7.24.8
│ ├─@babel/helper-module-imports@7.24.7
│ ├─v:@babel/helper-module-transforms@7.25.2
│ ├─@babel/helper-optimise-call-expression@7.24.7
│ ├─@babel/helper-plugin-utils@7.24.8
│ ├─v:@babel/helper-remap-async-to-generator@7.25.0
│ ├─v:@babel/helper-replace-supers@7.25.0
│ ├─@babel/helper-simple-access@7.24.7
│ ├─@babel/helper-skip-transparent-expression-wrappers@7.24.7
│ ├─@babel/helper-string-parser@7.24.8
│ ├─@babel/helper-validator-identifier@7.24.7
│ ├─@babel/helper-validator-option@7.24.8
│ ├─@babel/helper-wrap-function@7.25.0
│ ├─@babel/helpers@7.25.6
│ ├─@babel/highlight@7.24.7
│ │ ├─ansi-styles@3.2.1 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ ├─chalk@2.4.2 - filled by: chalk@4.1.2 at .→w:wh:mobile
│ │ ├─color-convert@1.9.3 - filled by: color-convert@2.0.1 at .→w:wh:mobile
│ │ ├─color-name@1.1.3 - filled by: color-name@1.1.4 at .→w:wh:mobile
│ │ ├─escape-string-regexp@1.0.5 - filled by: escape-string-regexp@4.0.0 at .→w:wh:mobile
│ │ ├─has-flag@3.0.0 - filled by: has-flag@4.0.0 at .→w:wh:mobile
│ │ └─supports-color@5.5.0 - filled by: supports-color@7.2.0 at .→w:wh:mobile
│ ├─@babel/parser@7.25.6
│ ├─v:@babel/plugin-proposal-async-generator-functions@7.20.7
│ ├─v:@babel/plugin-proposal-class-properties@7.18.6
│ ├─v:@babel/plugin-proposal-export-default-from@7.24.7
│ ├─v:@babel/plugin-proposal-nullish-coalescing-operator@7.18.6
│ ├─v:@babel/plugin-proposal-numeric-separator@7.18.6
│ ├─v:@babel/plugin-proposal-object-rest-spread@7.20.7
│ ├─v:@babel/plugin-proposal-optional-catch-binding@7.18.6
│ ├─v:@babel/plugin-proposal-optional-chaining@7.21.0
│ ├─v:@babel/plugin-syntax-async-generators@7.8.4
│ ├─v:@babel/plugin-syntax-dynamic-import@7.8.3
│ ├─v:@babel/plugin-syntax-export-default-from@7.24.7
│ ├─v:@babel/plugin-syntax-flow@7.24.7
│ ├─v:@babel/plugin-syntax-jsx@7.24.7
│ ├─v:@babel/plugin-syntax-nullish-coalescing-operator@7.8.3
│ ├─v:@babel/plugin-syntax-numeric-separator@7.10.4
│ ├─v:@babel/plugin-syntax-object-rest-spread@7.8.3
│ ├─v:@babel/plugin-syntax-optional-catch-binding@7.8.3
│ ├─v:@babel/plugin-syntax-optional-chaining@7.8.3
│ ├─v:@babel/plugin-syntax-private-property-in-object@7.14.5
│ ├─v:@babel/plugin-syntax-typescript@7.25.4
│ ├─v:@babel/plugin-transform-arrow-functions@7.24.7
│ ├─v:@babel/plugin-transform-async-to-generator@7.24.7
│ ├─v:@babel/plugin-transform-block-scoping@7.25.0
│ ├─v:@babel/plugin-transform-classes@7.25.4
│ ├─v:@babel/plugin-transform-computed-properties@7.24.7
│ ├─v:@babel/plugin-transform-destructuring@7.24.8
│ ├─v:@babel/plugin-transform-flow-strip-types@7.25.2
│ ├─v:@babel/plugin-transform-function-name@7.25.1
│ ├─v:@babel/plugin-transform-literals@7.25.2
│ ├─v:@babel/plugin-transform-modules-commonjs@7.24.8
│ ├─v:@babel/plugin-transform-named-capturing-groups-regex@7.24.7
│ ├─v:@babel/plugin-transform-parameters@7.24.7
│ ├─v:@babel/plugin-transform-private-methods@7.25.4
│ ├─v:@babel/plugin-transform-private-property-in-object@7.24.7
│ ├─v:@babel/plugin-transform-react-display-name@7.24.7
│ ├─v:@babel/plugin-transform-react-jsx@7.25.2
│ ├─v:@babel/plugin-transform-react-jsx-self@7.24.7
│ ├─v:@babel/plugin-transform-react-jsx-source@7.24.7
│ ├─v:@babel/plugin-transform-runtime@7.25.4
│ ├─v:@babel/plugin-transform-shorthand-properties@7.24.7
│ ├─v:@babel/plugin-transform-spread@7.24.7
│ ├─v:@babel/plugin-transform-sticky-regex@7.24.7
│ ├─v:@babel/plugin-transform-typescript@7.25.2
│ ├─v:@babel/plugin-transform-unicode-regex@7.24.7
│ ├─v:@babel/preset-flow@7.24.7
│ ├─v:@babel/preset-typescript@7.24.7
│ ├─v:@babel/register@7.24.6
│ ├─@babel/regjsgen@0.8.0
│ ├─@babel/runtime@7.25.6
│ ├─@babel/template@7.25.0
│ ├─@babel/traverse@7.25.6
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─@babel/types@7.25.6
│ ├─@hapi/hoek@9.3.0
│ ├─@hapi/topo@5.1.0
│ ├─@isaacs/cliui@8.0.2
│ │ ├─ansi-regex@6.1.0 - filled by: ansi-regex@5.0.1 at .→w:wh:mobile
│ │ ├─ansi-styles@6.2.1 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ ├─emoji-regex@9.2.2 - filled by: emoji-regex@8.0.0 at .→w:wh:mobile
│ │ ├─string-width@5.1.2 - filled by: string-width@4.2.3 at .→w:wh:mobile
│ │ ├─strip-ansi@7.1.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ │ └─wrap-ansi@8.1.0 - filled by: wrap-ansi@6.2.0 at .→w:wh:mobile
│ ├─@isaacs/ttlcache@1.4.1
│ ├─@jest/create-cache-key-function@29.7.0
│ ├─@jest/environment@29.7.0
│ ├─@jest/fake-timers@29.7.0
│ ├─@jest/schemas@29.6.3
│ ├─@jest/types@29.6.3
│ ├─@jridgewell/gen-mapping@0.3.5
│ ├─@jridgewell/resolve-uri@3.1.2
│ ├─@jridgewell/set-array@1.2.1
│ ├─@jridgewell/source-map@0.3.6
│ ├─@jridgewell/sourcemap-codec@1.5.0
│ ├─@jridgewell/trace-mapping@0.3.25
│ ├─@npmcli/agent@2.2.2
│ ├─@npmcli/fs@3.1.1
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@pkgjs/parseargs@0.11.0
│ ├─@react-native-community/cli@12.3.7
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-clean@12.3.7
│ ├─@react-native-community/cli-config@12.3.7
│ ├─@react-native-community/cli-debugger-ui@12.3.7
│ ├─@react-native-community/cli-doctor@12.3.7
│ │ ├─ansi-regex@4.1.1 - filled by: ansi-regex@5.0.1 at .→w:wh:mobile
│ │ ├─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ │ └─strip-ansi@5.2.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-hermes@12.3.7
│ ├─@react-native-community/cli-platform-android@12.3.7
│ ├─@react-native-community/cli-platform-ios@12.3.7
│ ├─@react-native-community/cli-plugin-metro@12.3.7
│ ├─@react-native-community/cli-server-api@12.3.7
│ ├─@react-native-community/cli-tools@12.3.7
│ │ ├─find-up@5.0.0 - filled by: find-up@4.1.0 at .→w:wh:mobile
│ │ └─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─@react-native-community/cli-types@12.3.7
│ ├─@react-native/assets-registry@0.73.1
│ ├─@react-native/babel-plugin-codegen@0.73.4
│ ├─v:@react-native/babel-preset@0.73.21
│ ├─v:@react-native/codegen@0.73.3
│ │ └─mkdirp@0.5.6 - filled by: mkdirp@1.0.4 at .→w:wh:mobile
│ ├─@react-native/community-cli-plugin@0.73.18
│ ├─@react-native/debugger-frontend@0.73.3
│ ├─@react-native/dev-middleware@0.73.8
│ │ ├─open@7.4.2 - filled by: open@6.4.0 at .→w:wh:mobile
│ │ └─v:ws@6.2.3 - filled by: ws@7.5.10 at .→w:wh:mobile
│ ├─@react-native/gradle-plugin@0.73.4
│ ├─@react-native/js-polyfills@0.73.1
│ ├─v:@react-native/metro-babel-transformer@0.73.15
│ │ ├─hermes-estree@0.15.0 - filled by: hermes-estree@0.23.1 at .→w:wh:mobile
│ │ └─hermes-parser@0.15.0 - filled by: hermes-parser@0.23.1 at .→w:wh:mobile
│ ├─@react-native/normalize-colors@0.73.2
│ ├─v:@react-native/virtualized-lists@0.73.4
│ ├─@sideway/address@4.1.5
│ ├─@sideway/formula@3.0.1
│ ├─@sideway/pinpoint@2.0.0
│ ├─@sinclair/typebox@0.27.8
│ ├─@sinonjs/commons@3.0.1
│ ├─@sinonjs/fake-timers@10.3.0
│ ├─@tanstack/query-core@4.36.1
│ ├─v:@tanstack/react-query@4.36.1
│ ├─@types/istanbul-lib-coverage@2.0.6
│ ├─@types/istanbul-lib-report@3.0.3
│ ├─@types/istanbul-reports@3.0.4
│ ├─@types/node@22.5.4
│ ├─@types/stack-utils@2.0.3
│ ├─@types/yargs@17.0.33
│ ├─@types/yargs-parser@21.0.3
│ ├─abbrev@2.0.0
│ ├─abort-controller@3.0.0
│ ├─accepts@1.3.8
│ ├─acorn@8.12.1
│ ├─agent-base@7.1.1
│ │ └─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─aggregate-error@3.1.0
│ ├─anser@1.4.10
│ ├─ansi-fragments@0.2.1
│ │ ├─ansi-regex@4.1.1 - filled by: ansi-regex@5.0.1 at .→w:wh:mobile
│ │ └─strip-ansi@5.2.0 - filled by: strip-ansi@6.0.1 at .→w:wh:mobile
│ ├─ansi-regex@5.0.1
│ ├─ansi-styles@4.3.0
│ ├─anymatch@3.1.3
│ ├─appdirsjs@1.2.7
│ ├─argparse@1.0.10
│ ├─asap@2.0.6
│ ├─ast-types@0.15.2
│ ├─astral-regex@1.0.0
│ ├─async-limiter@1.0.1
│ ├─v:babel-core@7.0.0-bridge.0
│ ├─v:babel-plugin-polyfill-corejs2@0.4.11
│ ├─v:babel-plugin-polyfill-corejs3@0.10.6
│ ├─v:babel-plugin-polyfill-regenerator@0.6.2
│ ├─babel-plugin-transform-flow-enums@0.0.2
│ ├─balanced-match@1.0.2
│ ├─base64-js@1.5.1
│ ├─bl@4.1.0
│ ├─brace-expansion@1.1.11
│ ├─braces@3.0.3
│ ├─browserslist@4.23.3
│ ├─bser@2.1.1
│ ├─buffer@5.7.1
│ ├─buffer-from@1.1.2
│ ├─bytes@3.0.0
│ ├─cacache@18.0.4
│ │ ├─brace-expansion@2.0.1 - filled by: brace-expansion@1.1.11 at .→w:wh:mobile
│ │ ├─glob@10.4.5 - filled by: glob@7.2.3 at .→w:wh:mobile
│ │ └─minimatch@9.0.5 - filled by: minimatch@3.1.2 at .→w:wh:mobile
│ ├─caller-callsite@2.0.0
│ ├─caller-path@2.0.0
│ ├─callsites@2.0.0
│ ├─camelcase@5.3.1
│ ├─caniuse-lite@1.0.30001660
│ ├─chalk@4.1.2
│ ├─chownr@2.0.0
│ ├─chrome-launcher@0.15.2
│ ├─chromium-edge-launcher@1.0.0
│ ├─ci-info@2.0.0
│ ├─clean-stack@2.2.0
│ ├─cli-cursor@3.1.0
│ ├─cli-spinners@2.9.2
│ ├─cliui@6.0.0
│ ├─clone@1.0.4
│ ├─clone-deep@4.0.1
│ ├─color-convert@2.0.1
│ ├─color-name@1.1.4
│ ├─colorette@1.4.0
│ ├─command-exists@1.2.9
│ ├─commander@9.5.0
│ ├─commondir@1.0.1
│ ├─compressible@2.0.18
│ │ └─mime-db@1.53.0 - filled by: mime-db@1.52.0 at .→w:wh:mobile
│ ├─compression@1.7.4
│ ├─concat-map@0.0.1
│ ├─connect@3.7.0
│ ├─convert-source-map@2.0.0
│ ├─core-js-compat@3.38.1
│ ├─core-util-is@1.0.3
│ ├─cosmiconfig@5.2.1
│ ├─cross-spawn@7.0.3
│ ├─dayjs@1.11.13
│ ├─v:debug@2.6.9
│ │ └─ms@2.0.0 - filled by: ms@2.1.3 at .→w:wh:mobile
│ ├─decamelize@1.2.0
│ ├─deepmerge@4.3.1
│ ├─defaults@1.0.4
│ ├─denodeify@1.2.1
│ ├─depd@2.0.0
│ ├─deprecated-react-native-prop-types@5.0.0
│ ├─destroy@1.2.0
│ ├─eastasianwidth@0.2.0
│ ├─ee-first@1.1.1
│ ├─electron-to-chromium@1.5.18
│ ├─emoji-regex@8.0.0
│ ├─encodeurl@1.0.2
│ ├─encoding@0.1.13
│ ├─env-paths@2.2.1
│ ├─envinfo@7.13.0
│ ├─err-code@2.0.3
│ ├─error-ex@1.3.2
│ ├─error-stack-parser@2.1.4
│ ├─errorhandler@1.5.1
│ ├─escalade@3.2.0
│ ├─escape-html@1.0.3
│ ├─escape-string-regexp@4.0.0
│ ├─esprima@4.0.1
│ ├─etag@1.8.1
│ ├─event-target-shim@5.0.1
│ ├─execa@5.1.1
│ ├─exponential-backoff@3.1.1
│ ├─fast-xml-parser@4.5.0
│ ├─fb-watchman@2.0.2
│ ├─fill-range@7.1.1
│ ├─finalhandler@1.1.2
│ │ ├─on-finished@2.3.0 - filled by: on-finished@2.4.1 at .→w:wh:mobile
│ │ └─statuses@1.5.0 - filled by: statuses@2.0.1 at .→w:wh:mobile
│ ├─find-cache-dir@2.1.0
│ ├─find-up@4.1.0
│ │ ├─locate-path@5.0.0 - filled by: locate-path@6.0.0 at .→w:wh:mobile
│ │ └─p-locate@4.1.0 - filled by: p-locate@5.0.0 at .→w:wh:mobile
│ ├─flow-enums-runtime@0.0.6
│ ├─flow-parser@0.206.0
│ ├─foreground-child@3.3.0
│ │ └─signal-exit@4.1.0 - filled by: signal-exit@3.0.7 at .→w:wh:mobile
│ ├─fresh@0.5.2
│ ├─fs-extra@8.1.0
│ ├─fs-minipass@3.0.3
│ ├─fs.realpath@1.0.0
│ ├─fsevents@optional!builtin<compat/fsevents>::version=2.3.3&hash=df0bf1
│ ├─function-bind@1.1.2
│ ├─gensync@1.0.0-beta.2
│ ├─get-caller-file@2.0.5
│ ├─get-stream@6.0.1
│ ├─glob@7.2.3
│ ├─globals@11.12.0
│ ├─graceful-fs@4.2.11
│ ├─has-flag@4.0.0
│ ├─hasown@2.0.2
│ ├─hermes-estree@0.23.1
│ ├─hermes-parser@0.23.1
│ ├─hermes-profile-transformer@0.0.6
│ │ └─source-map@0.7.4 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─http-cache-semantics@4.1.1
│ ├─http-errors@2.0.0
│ ├─http-proxy-agent@7.0.2
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─https-proxy-agent@7.0.5
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─human-signals@2.1.0
│ ├─iconv-lite@0.6.3
│ ├─ieee754@1.2.1
│ ├─image-size@1.1.1
│ ├─import-fresh@2.0.0
│ ├─imurmurhash@0.1.4
│ ├─indent-string@4.0.0
│ ├─inflight@1.0.6
│ ├─inherits@2.0.4
│ ├─invariant@2.2.4
│ ├─ip-address@9.0.5
│ │ └─sprintf-js@1.1.3 - filled by: sprintf-js@1.0.3 at .→w:wh:mobile
│ ├─is-arrayish@0.2.1
│ ├─is-core-module@2.15.1
│ ├─is-directory@0.3.1
│ ├─is-docker@2.2.1
│ ├─is-fullwidth-code-point@2.0.0
│ ├─is-interactive@1.0.0
│ ├─is-lambda@1.0.1
│ ├─is-number@7.0.0
│ ├─is-plain-object@2.0.4
│ ├─is-stream@2.0.1
│ ├─is-unicode-supported@0.1.0
│ ├─is-wsl@2.2.0
│ ├─isarray@1.0.0
│ ├─isexe@2.0.0
│ ├─isobject@3.0.1
│ ├─jackspeak@3.4.3
│ ├─jest-environment-node@29.7.0
│ ├─jest-get-type@29.6.3
│ ├─jest-message-util@29.7.0
│ │ ├─ansi-styles@5.2.0 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ └─pretty-format@29.7.0 - filled by: pretty-format@26.6.2 at .→w:wh:mobile
│ ├─jest-mock@29.7.0
│ ├─jest-util@29.7.0
│ │ ├─ci-info@3.9.0 - filled by: ci-info@2.0.0 at .→w:wh:mobile
│ ├─jest-validate@29.7.0
│ │ ├─ansi-styles@5.2.0 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ ├─camelcase@6.3.0 - filled by: camelcase@5.3.1 at .→w:wh:mobile
│ │ └─pretty-format@29.7.0 - filled by: pretty-format@26.6.2 at .→w:wh:mobile
│ ├─jest-worker@29.7.0
│ │ └─supports-color@8.1.1 - filled by: supports-color@7.2.0 at .→w:wh:mobile
│ ├─joi@17.13.3
│ ├─js-tokens@4.0.0
│ ├─js-yaml@3.14.1
│ ├─jsbn@1.1.0
│ ├─jsc-android@250231.0.0
│ ├─jsc-safe-url@0.2.4
│ ├─v:jscodeshift@0.14.0
│ │ ├─flow-parser@0.245.2 - filled by: flow-parser@0.206.0 at .→w:wh:mobile
│ ├─jsesc@2.5.2
│ ├─json-parse-better-errors@1.0.2
│ ├─json5@2.2.3
│ ├─jsonfile@4.0.0
│ ├─kind-of@6.0.3
│ ├─kleur@3.0.3
│ ├─leven@3.1.0
│ ├─lighthouse-logger@1.4.2
│ ├─locate-path@6.0.0
│ ├─lodash.debounce@4.0.8
│ ├─lodash.throttle@4.1.1
│ ├─log-symbols@4.1.0
│ ├─logkitty@0.7.1
│ │ └─yargs@15.4.1 - filled by: yargs@17.7.2 at .→w:wh:mobile
│ ├─loose-envify@1.4.0
│ ├─lru-cache@10.4.3
│ ├─make-dir@2.1.0
│ │ └─semver@5.7.2 - filled by: semver@6.3.1 at .→w:wh:mobile
│ ├─make-fetch-happen@13.0.1
│ ├─makeerror@1.0.12
│ ├─marky@1.2.5
│ ├─memoize-one@5.2.1
│ ├─merge-stream@2.0.0
│ ├─metro@0.80.12
│ ├─metro-babel-transformer@0.80.12
│ ├─metro-cache@0.80.12
│ ├─metro-cache-key@0.80.12
│ ├─metro-config@0.80.12
│ ├─metro-core@0.80.12
│ ├─metro-file-map@0.80.12
│ ├─metro-minify-terser@0.80.12
│ ├─metro-resolver@0.80.12
│ ├─metro-runtime@0.80.12
│ ├─metro-source-map@0.80.12
│ ├─metro-symbolicate@0.80.12
│ ├─metro-transform-plugins@0.80.12
│ ├─metro-transform-worker@0.80.12
│ ├─micromatch@4.0.8
│ ├─mime@2.6.0
│ ├─mime-db@1.52.0
│ ├─mime-types@2.1.35
│ ├─mimic-fn@2.1.0
│ ├─minimatch@3.1.2
│ ├─minimist@1.2.8
│ ├─minipass@7.1.2
│ ├─minipass-collect@2.0.1
│ ├─minipass-fetch@3.0.5
│ ├─minipass-flush@1.0.5
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minipass-pipeline@1.2.4
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minipass-sized@1.0.3
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─minizlib@2.1.2
│ │ ├─minipass@3.3.6 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─mkdirp@1.0.4
│ ├─w:mobile
│ ├─ms@2.1.3
│ ├─negotiator@0.6.3
│ ├─neo-async@2.6.2
│ ├─nocache@3.0.4
│ ├─node-abort-controller@3.1.1
│ ├─node-dir@0.1.17
│ ├─v:node-fetch@2.7.0
│ ├─node-gyp@10.2.0
│ │ ├─brace-expansion@2.0.1 - filled by: brace-expansion@1.1.11 at .→w:wh:mobile
│ │ ├─glob@10.4.5 - filled by: glob@7.2.3 at .→w:wh:mobile
│ │ ├─isexe@3.1.1 - filled by: isexe@2.0.0 at .→w:wh:mobile
│ │ ├─minimatch@9.0.5 - filled by: minimatch@3.1.2 at .→w:wh:mobile
│ │ ├─semver@7.6.3 - filled by: semver@6.3.1 at .→w:wh:mobile
│ │ └─which@4.0.0 - filled by: which@2.0.2 at .→w:wh:mobile
│ ├─node-int64@0.4.0
│ ├─node-releases@2.0.18
│ ├─node-stream-zip@1.15.0
│ ├─nopt@7.2.1
│ ├─normalize-path@3.0.0
│ ├─npm-run-path@4.0.1
│ ├─nullthrows@1.1.1
│ ├─ob1@0.80.12
│ ├─object-assign@4.1.1
│ ├─on-finished@2.4.1
│ ├─on-headers@1.0.2
│ ├─once@1.4.0
│ ├─onetime@5.1.2
│ ├─open@6.4.0
│ │ ├─is-wsl@1.1.0 - filled by: is-wsl@2.2.0 at .→w:wh:mobile
│ ├─ora@5.4.1
│ ├─p-limit@2.3.0
│ ├─p-locate@5.0.0
│ │ ├─p-limit@3.1.0 - filled by: p-limit@2.3.0 at .→w:wh:mobile
│ ├─p-map@4.0.0
│ ├─p-try@2.2.0
│ ├─package-json-from-dist@1.0.0
│ ├─parse-json@4.0.0
│ ├─parseurl@1.3.3
│ ├─path-exists@4.0.0
│ ├─path-is-absolute@1.0.1
│ ├─path-key@3.1.1
│ ├─path-parse@1.0.7
│ ├─path-scurry@1.11.1
│ ├─picocolors@1.1.0
│ ├─picomatch@2.3.1
│ ├─pify@4.0.1
│ ├─pirates@4.0.6
│ ├─pkg-dir@3.0.0
│ │ ├─find-up@3.0.0 - filled by: find-up@4.1.0 at .→w:wh:mobile
│ │ ├─locate-path@3.0.0 - filled by: locate-path@6.0.0 at .→w:wh:mobile
│ │ ├─p-locate@3.0.0 - filled by: p-locate@5.0.0 at .→w:wh:mobile
│ │ ├─path-exists@3.0.0 - filled by: path-exists@4.0.0 at .→w:wh:mobile
│ ├─pretty-format@26.6.2
│ │ ├─@jest/types@26.6.2 - filled by: @jest/types@29.6.3 at .→w:wh:mobile
│ │ ├─@types/yargs@15.0.19 - filled by: @types/yargs@17.0.33 at .→w:wh:mobile
│ │ └─react-is@17.0.2 - filled by: react-is@18.3.1 at .→w:wh:mobile
│ ├─proc-log@4.2.0
│ ├─process-nextick-args@2.0.1
│ ├─promise@8.3.0
│ ├─promise-retry@2.0.1
│ ├─prompts@2.4.2
│ ├─prop-types@15.8.1
│ │ └─react-is@16.13.1 - filled by: react-is@18.3.1 at .→w:wh:mobile
│ ├─queue@6.0.2
│ ├─range-parser@1.2.1
│ ├─react@18.2.0
│ ├─react-devtools-core@4.28.5
│ ├─react-is@18.3.1
│ ├─v:react-native@0.73.9
│ │ ├─mkdirp@0.5.6 - filled by: mkdirp@1.0.4 at .→w:wh:mobile
│ │ ├─regenerator-runtime@0.13.11 - filled by: regenerator-runtime@0.14.1 at .→w:wh:mobile
│ │ └─v:ws@6.2.3 - filled by: ws@7.5.10 at .→w:wh:mobile
│ ├─react-refresh@0.14.2
│ ├─v:react-shallow-renderer@16.15.0
│ ├─readable-stream@3.6.2
│ ├─readline@1.3.0
│ ├─recast@0.21.5
│ │ └─source-map@0.6.1 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─regenerate@1.4.2
│ ├─regenerate-unicode-properties@10.1.1
│ ├─regenerator-runtime@0.14.1
│ ├─regexpu-core@5.3.2
│ ├─regjsparser@0.9.1
│ │ ├─jsesc@0.5.0 - filled by: jsesc@2.5.2 at .→w:wh:mobile
│ ├─require-directory@2.1.1
│ ├─require-main-filename@2.0.0
│ ├─resolve@optional!builtin<compat/resolve>::version=1.22.8&hash=c3c19d
│ ├─resolve-from@3.0.0
│ ├─restore-cursor@3.1.0
│ ├─retry@0.12.0
│ ├─rimraf@3.0.2
│ ├─safe-buffer@5.1.2
│ ├─safer-buffer@2.1.2
│ ├─scheduler@0.24.0-canary-efb381bbf-20230505
│ ├─semver@6.3.1
│ ├─send@0.18.0
│ │ ├─mime@1.6.0 - filled by: mime@2.6.0 at .→w:wh:mobile
│ ├─serialize-error@2.1.0
│ ├─serve-static@1.16.0
│ ├─set-blocking@2.0.0
│ ├─setprototypeof@1.2.0
│ ├─shallow-clone@3.0.1
│ ├─shebang-command@2.0.0
│ ├─shebang-regex@3.0.0
│ ├─shell-quote@1.8.1
│ ├─signal-exit@3.0.7
│ ├─sisteransi@1.0.5
│ ├─slash@3.0.0
│ ├─slice-ansi@2.1.0
│ │ ├─ansi-styles@3.2.1 - filled by: ansi-styles@4.3.0 at .→w:wh:mobile
│ │ ├─color-convert@1.9.3 - filled by: color-convert@2.0.1 at .→w:wh:mobile
│ │ ├─color-name@1.1.3 - filled by: color-name@1.1.4 at .→w:wh:mobile
│ ├─smart-buffer@4.2.0
│ ├─socks@2.8.3
│ ├─socks-proxy-agent@8.0.4
│ │ ├─v:debug@4.3.7 - filled by: debug@2.6.9 at .→w:wh:mobile
│ ├─source-map@0.5.7
│ ├─source-map-support@0.5.21
│ │ ├─source-map@0.6.1 - filled by: source-map@0.5.7 at .→w:wh:mobile
│ ├─sprintf-js@1.0.3
│ ├─ssri@10.0.6
│ ├─stack-utils@2.0.6
│ │ ├─escape-string-regexp@2.0.0 - filled by: escape-string-regexp@4.0.0 at .→w:wh:mobile
│ ├─stackframe@1.3.4
│ ├─stacktrace-parser@0.1.10
│ ├─statuses@2.0.1
│ ├─string-width@4.2.3
│ │ ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ ├─a:string-width-cjs:string-width@4.2.3
│ │ ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ │ └─string-width@4.2.3 - self-reference
│ │   ├─is-fullwidth-code-point@3.0.0 - filled by: is-fullwidth-code-point@2.0.0 at .→w:wh:mobile
│ ├─string_decoder@1.3.0
│ │ ├─safe-buffer@5.2.1 - filled by: safe-buffer@5.1.2 at .→w:wh:mobile
│ ├─strip-ansi@6.0.1
│ ├─a:strip-ansi-cjs:strip-ansi@6.0.1
│ │ └─strip-ansi@6.0.1 - self-reference
│ ├─strip-final-newline@2.0.0
│ ├─strnum@1.0.5
│ ├─sudo-prompt@9.2.1
│ ├─supports-color@7.2.0
│ ├─supports-preserve-symlinks-flag@1.0.0
│ ├─tar@6.2.1
│ │ ├─fs-minipass@2.1.0 - filled by: fs-minipass@3.0.3 at .→w:wh:mobile
│ │ │ └─minipass@3.3.6 - filled by minipass@5.0.0 at w:wh:mobile
│ │ ├─minipass@5.0.0 - filled by: minipass@7.1.2 at .→w:wh:mobile
│ ├─temp@0.8.4
│ │ ├─rimraf@2.6.3 - filled by: rimraf@3.0.2 at .→w:wh:mobile
│ ├─temp-dir@2.0.0
│ ├─terser@5.32.0
│ │ ├─commander@2.20.3 - filled by: commander@9.5.0 at .→w:wh:mobile
│ ├─throat@5.0.0
│ ├─through2@2.0.5
│ │ ├─readable-stream@2.3.8 - filled by: readable-stream@3.6.2 at .→w:wh:mobile
│ │ ├─string_decoder@1.1.1 - filled by: string_decoder@1.3.0 at .→w:wh:mobile
│ ├─tmpl@1.0.5
│ ├─to-fast-properties@2.0.0
│ ├─to-regex-range@5.0.1
│ ├─toidentifier@1.0.1
│ ├─tr46@0.0.3
│ ├─tslib@2.7.0
│ ├─type-detect@4.0.8
│ ├─type-fest@0.7.1
│ ├─undici-types@6.19.8
│ ├─unicode-canonical-property-names-ecmascript@2.0.0
│ ├─unicode-match-property-ecmascript@2.0.0
│ ├─unicode-match-property-value-ecmascript@2.1.0
│ ├─unicode-property-aliases-ecmascript@2.1.0
│ ├─unique-filename@3.0.0
│ ├─unique-slug@4.0.0
│ ├─universalify@0.1.2
│ ├─unpipe@1.0.0
│ ├─v:update-browserslist-db@1.1.0
│ ├─v:use-sync-external-store@1.2.2
│ ├─util-deprecate@1.0.2
│ ├─utils-merge@1.0.1
│ ├─vary@1.1.2
│ ├─vlq@1.0.1
│ ├─walker@1.0.8
│ ├─wcwidth@1.0.1
│ ├─webidl-conversions@3.0.1
│ ├─whatwg-fetch@3.6.20
│ ├─whatwg-url@5.0.0
│ ├─which@2.0.2
│ ├─which-module@2.0.1
│ ├─wrap-ansi@6.2.0
│ ├─a:wrap-ansi-cjs:wrap-ansi@7.0.0
│ │ └─wrap-ansi@7.0.0 - self-reference
│ ├─wrappy@1.0.2
│ ├─write-file-atomic@2.4.3
│ ├─v:ws@7.5.10
│ ├─xtend@4.0.2
│ ├─y18n@4.0.3
│ ├─yallist@4.0.0
│ ├─yaml@2.5.1
│ ├─yargs@17.7.2
│ │ ├─cliui@8.0.1 - filled by: cliui@6.0.0 at .→w:wh:mobile
│ │ ├─wrap-ansi@7.0.0 - filled by: wrap-ansi@6.2.0 at .→w:wh:mobile
│ │ ├─y18n@5.0.8 - filled by: y18n@4.0.3 at .→w:wh:mobile
│ │ └─yargs-parser@21.1.1 - filled by: yargs-parser@18.1.3 at .→w:wh:mobile
│ ├─yargs-parser@18.1.3
│ └─yocto-queue@0.1.0
├─react@18.3.1
├─w:web
├─w:wh:web
├─w:web-common
├─w:wh:web-common

    at yIe (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:571:10362)
    at FB (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:579:261)
    at aj.finalizeInstall (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:662:25971)
    at async t.linkEverything (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:210:17393)
    at async /Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:213:4509
    at async Rt.startSectionPromise (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:176:2828)
    at async t.install (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:213:4315)
    at async /Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:447:18229
    at async Rt.start (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:176:1995)
    at async lE.execute (/Users/santeri/.cache/node/corepack/v1/yarn/4.4.1/yarn.js:447:18086)
➤ YN0000: └ Completed
➤ YN0000: · Failed with errors in 0s 356ms
```
