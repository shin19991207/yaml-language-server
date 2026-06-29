![CI](https://github.com/redhat-developer/yaml-language-server/workflows/CI/badge.svg) [![version](https://img.shields.io/npm/v/yaml-language-server.svg)](https://www.npmjs.com/package/yaml-language-server) [![Coverage Status](https://coveralls.io/repos/github/redhat-developer/yaml-language-server/badge.svg?branch=main)](https://coveralls.io/github/redhat-developer/yaml-language-server?branch=main)

# YAML Language Server

`yaml-language-server` provides YAML language features over the Language Server Protocol (LSP), including validation, completion, hover, formatting, document symbols, and schema-based intelligence.

Starting from version `1.0.0`, the language server uses [eemeli/yaml](https://github.com/eemeli/yaml) as its YAML parser, which strictly enforces the specified YAML spec version. The default YAML spec version is `1.2`. Set `yaml.yamlVersion` to `1.1` for compatibility with older YAML files.

Schema validation supports JSON Schema `draft-04`, `draft-07`, `2019-09`, and `2020-12`.

## Features

1. **YAML validation**:
   - Detects whether the entire file is valid YAML
   - Reports diagnostics such as:
     - Node is not found
     - Node has an invalid key node type
     - Node has an invalid type
     - Node is not a valid child node
     - Node is an additional property of its parent
2. **Document symbols**:
   - Provides document symbols and hierarchical document symbols for YAML nodes
3. **Completion**:
   - Completes YAML keys, values, and structure based on the associated schema
   - Completes scalar nodes with schema defaults when defaults are available
4. **Hover support**:
   - Shows schema descriptions for YAML nodes when descriptions are available
   - Shows anchor information when `yaml.hoverAnchor` is enabled
   - Shows schema source information when `yaml.hoverSchemaSource` is enabled
5. **Formatting**:
   - Formats YAML documents
   - Supports on-type formatting on newline, including automatic indentation for mappings and array items

Completion and hover content are schema-driven. Configure schemas with a modeline, `yaml.schemas`, schema association notifications, or Schema Store.

## Language Server Settings

Settings are supplied through LSP configuration. Setting names match the `yaml.*` configuration used by common integrations.

- `yaml.yamlVersion`: Set default YAML spec version (`1.2` or `1.1`). Defaults to `1.2`.
- `yaml.maxItemsComputed`: The maximum number of outline symbols and folding regions computed (limited for performance reasons). Defaults to `5000`.
- `yaml.format.enable`: Enable/disable default YAML formatter. Defaults to `true`.
- `yaml.format.singleQuote`: Use single quotes instead of double quotes. Defaults to `false`.
- `yaml.format.bracketSpacing`: Print spaces between brackets in objects. Defaults to `true`.
- `yaml.format.proseWrap`: Control prose wrapping behavior. `always`: wrap prose if it exceeds the print width, `never`: never wrap the prose, `preserve`: wrap prose as-is. Defaults to `preserve`.
- `yaml.format.printWidth`: Specify the line length that the printer will wrap on. Defaults to `80`.
- `yaml.format.trailingComma`: Specify if trailing commas should be used in JSON-like segments of the YAML. Defaults to `true`.
- `yaml.validate`: Enable/disable validation feature. Defaults to `true`.
- `yaml.hover`: Enable/disable hover. Defaults to `true`.
- `yaml.hoverAnchor`: Enable/disable hover feature for anchors. Defaults to `true`.
- `yaml.hoverSchemaSource`: Enable/disable showing the schema source in hover tooltips. Defaults to `true`.
- `yaml.completion`: Enable/disable autocompletion. Defaults to `true`.
- `yaml.disableDefaultProperties`: Disable adding not required properties with default values into completion text. Defaults to `false`.
- `yaml.suggest.parentSkeletonSelectedFirst`: If true, the user must select some parent skeleton first before autocompletion starts to suggest the rest of the properties. When the YAML object is not empty, autocompletion ignores this setting and returns all properties and skeletons. Defaults to `false`.
- `yaml.schemas`: Associate schemas with files using glob patterns. See [Associating schemas](#associating-schemas) for details.
- `yaml.disableSchemaDetection`: Disable schema detection for YAML files matching the configured glob pattern or list of glob patterns. Modelines still apply.
- `yaml.schemaStore.enable`: When set to true, the YAML language server will pull in all available schemas from [JSON Schema Store](http://schemastore.org/). Defaults to `true`.
- `yaml.schemaStore.url`: URL of a schema store catalog to use when downloading schemas. Defaults to `https://www.schemastore.org/api/json/catalog.json`.
- `yaml.customTags`: Array of custom tags that the parser will validate against. It has three ways to be used. A tag without a type, such as "!Ref", is treated as a scalar tag. A tag with a node type, such as "!Ref sequence", specifies the YAML node type that the tag is written on. A tag with a node type and return type, such as "!FindInMap sequence:string", also specifies the schema type that the tagged value evaluates to. Supported node types are scalar, sequence, and mapping. Supported return types are string, number, integer, boolean, null, array, and object. The return type aliases scalar, sequence, and mapping are accepted as string, array, and object. See [Adding custom tags](#adding-custom-tags) for usage details.
- `yaml.disableAdditionalProperties`: Globally set `additionalProperties` to `false` for all objects. When enabled, no extra properties are allowed in YAML objects beyond those defined in the schema. Defaults to `false`.
- `yaml.kubernetesCRDStore.enable`: Enable/disable validation of Kubernetes custom resources using schemas from well-known Custom Resource Definitions (CRDs). Defaults to `true`.
- `yaml.kubernetesCRDStore.url`: The base URL for fetching well-known Custom Resource Definition (CRD) schemas. Defaults to `https://raw.githubusercontent.com/datreeio/CRDs-catalog/main`.
- `yaml.kubernetesVersion`: Kubernetes version used to build the schema URL when `yaml.schemas` maps files to the `kubernetes` keyword. If omitted, the extension falls back to a predefined default Kubernetes version.
- `yaml.style.flowMapping`: Control flow style mappings. Forbids flow style mappings if set to `forbid`. Defaults to `allow`.
- `yaml.style.flowSequence`: Control flow style sequences. Forbids flow style sequences if set to `forbid`. Defaults to `allow`.
- `yaml.keyOrdering`: Enforces alphabetical ordering of keys in mappings when set to `true`. Defaults to `false`.
- `http.proxy`: The URL of the proxy server that will be used when attempting to download a schema. If it is not set or it is undefined, no proxy server will be used.
- `http.proxyStrictSSL`: If true, the proxy server certificate should be verified against the list of supplied CAs. Defaults to `false`.
- `[yaml].editor.tabSize`: Number of spaces to use for YAML indentation. When provided, this setting is used for generated completion text. Defaults to `2`.
- `editor.tabSize`: Fallback indentation size when no YAML-specific tab size is provided. Defaults to `2`.
- `[yaml].editor.formatOnType`: Client/editor setting that controls whether on-type formatting requests are sent for YAML files. When enabled by the client, `yaml-language-server` can format YAML as the user types, such as adjusting indentation after a newline.

## Associating Schemas

YAML Language Server uses [JSON Schema](https://json-schema.org/) to understand the shape of a YAML file. It supports JSON Schema Draft 4, Draft 7, Draft 2019-09, and Draft 2020-12. Schema definitions can be written in JSON (`.json`) or YAML (`.yaml`).

You can associate schemas with YAML files by adding a modeline or configuring `yaml.schemas` through your LSP client. When `yaml.schemaStore.enable` is enabled, as it is by default, YAML Language Server also associates schemas from [SchemaStore](https://www.schemastore.org/) with files that match their configured patterns.

### Using a Modeline

You can specify a JSON schema directly in a YAML file with a modeline comment. If a relative path is specified, it is resolved from the YAML file location, not from the workspace root.

```yaml
# yaml-language-server: $schema=<urlToTheSchema>
```

The IntelliJ-compatible `$schema` comment format is also supported:

```yaml
# $schema: <urlOrPathToTheSchema>
```

### Using `yaml.schemas`

The `yaml.schemas` setting maps schemas to file patterns using key-value pairs:
* **Key**: Schema URI, local file path, or the `kubernetes` keyword
* **Value**: A glob pattern or array of glob patterns

#### Remote Schemas

Associate an online schema with YAML files:

```json
{
  "yaml.schemas": {
    "https://getcomposer.org/schema.json": "composer.yaml",
    "https://example.com/api-schema.json": ["api/*.yml", "api/*.yaml"]
  }
}
```

#### Local Schemas

Local schema paths can be absolute paths, file URIs, or relative paths. In a single-folder workspace, relative paths are resolved from the workspace root.

On macOS or Linux:

```json
{
  "yaml.schemas": {
    "/home/user/custom_schema.json": "someFilePattern.yaml",
    "/home/user/custom_schema.yaml": "anotherPattern.yaml",
    "../relative/path/schema.json": ["filePattern1.yaml", "filePattern2.yaml"]
  }
}
```

On Windows:

```json
{
  "yaml.schemas": {
    "C:\\Users\\user\\Documents\\custom_schema.json": "someFilePattern.yaml",
    "file:///C:/Users/user/Documents/custom_schema.yaml": "anotherPattern.yaml",
    "../relative/path/schema.json": ["filePattern1.yaml", "filePattern2.yaml"]
  }
}
```

**Multi-Root Workspaces**

In a multi-root workspace, prefix local schema paths with the workspace folder that contains the schema.

Suppose you have a multi-root workspace that contains two workspace folders, `My_first_project` and `My_second_project`:

```shell
My_first_project/
├── test.yaml
└── my_schema.json
My_second_project/
├── test2.yaml
└── my_schema2.json
```

Use the workspace folder name at the start of each schema path key:

```json
{
  "yaml.schemas": {
    "My_first_project/my_schema.json": "test.yaml",
    "My_second_project/my_schema2.json": "test2.yaml"
  }
}
```

#### Using the Kubernetes Schema Keyword

Use the reserved `kubernetes` keyword to validate Kubernetes YAML files. The language server automatically resolves it to the appropriate versioned Kubernetes schema URL based on your `yaml.kubernetesVersion` setting (or the default version).

```json
{
  "yaml.schemas": {
    "kubernetes": "k8s/*.yaml"
  }
}
```

**Specifying Kubernetes Version**

Use `yaml.kubernetesVersion` to choose the Kubernetes version used for the generated schema URL.

```json
{
  "yaml.kubernetesVersion": "1.36.1",
  "yaml.schemas": {
    "kubernetes": "k8s/*.yaml"
  }
}
```

### Schema Association Notifications

Integrations can provide schema associations through the `json/schemaAssociations` notification. These associations are useful when an integration owns a YAML file type and wants the server to apply a specific schema automatically.

### Schema Priority

When multiple schema sources could apply to the same file, the language server uses this priority order from highest to lowest:

1. Modeline
2. Custom schema provider API
3. `yaml.disableSchemaDetection`
4. `yaml.schemas`
5. Schema association notification
6. Schema Store

## Suppressing Diagnostics

To hide diagnostics for a specific YAML line, add a suppression comment immediately before that line. To disable schema validation for an entire file, see [Disabling Schema Validation](#disabling-schema-validation).

### Suppress All Diagnostics on a Line

Add `# yaml-language-server-disable` immediately before the line that produces the diagnostic:

```yaml
# yaml-language-server-disable
version: 123
```

### Suppress Matching Diagnostics

Add one or more comma-separated diagnostic message substrings after `# yaml-language-server-disable`. Only diagnostics whose messages contain a matching substring are suppressed; the rest are still reported. Matching is case-insensitive.

Single substring:

```yaml
# yaml-language-server-disable Incorrect type
version: 123
```

Multiple substrings:

```yaml
# yaml-language-server-disable Incorrect type, not accepted
version: 123
```

The substrings are matched against the diagnostic message text reported by the language server.

## Disabling Schema Validation

Disabling schema validation stops schema-based diagnostics. The file is still parsed as YAML, so YAML syntax errors can still be reported.

### Using a Modeline

A modeline can disable schema validation for the current file by setting `$schema` to `none`:

```yaml
# yaml-language-server: $schema=none
```

### Using `yaml.disableSchemaDetection`

To prevent detected schemas from being applied to specific YAML files, configure `yaml.disableSchemaDetection` with one or more glob patterns. For matching files, schemas from `yaml.schemas`, schema association notifications, and Schema Store are ignored. Modelines still apply.

For one file pattern:

```yaml
yaml.disableSchemaDetection: "**/.github/workflows/*.yaml"
```

For multiple file patterns:

```yaml
yaml.disableSchemaDetection: ["some.yaml", "**/.github/workflows/*.yaml"]
```

## Adding Custom Tags

YAML custom tags extend the language with application-specific syntax. Configure custom tags with the `yaml.customTags` setting.

Each entry supports one of these formats:

- `!Tag`: Treats the tag as a scalar tag
- `!Tag nodeType`: Specifies the YAML node type for the tagged value
- `!Tag nodeType:returnType`: Specifies the YAML node type and the schema type used during validation

Supported node types are `scalar`, `sequence`, and `mapping`.

Supported return types are `string`, `number`, `integer`, `boolean`, `null`, `array`, and `object`. The aliases `scalar`, `sequence`, and `mapping` are also accepted as `string`, `array`, and `object`.

For example:

```yaml
yaml.customTags: [
  "!Scalar-example",
  "!Seq-example sequence",
  "!Mapping-example mapping",
  "!Seq-as-string-example sequence:string"
]
```

These tags can then be used in YAML files:

```yaml
some_key: !Scalar-example some_value
some_sequence: !Seq-example
  - some_seq_key_1: some_seq_value_1
  - some_seq_key_2: some_seq_value_2
some_mapping: !Mapping-example
  some_mapping_key_1: some_mapping_value_1
  some_mapping_key_2: some_mapping_value_2
some_string: !Seq-as-string-example
  - value_1
  - value_2
```

In the last example, `!Seq-as-string-example` is written on a YAML sequence, but schema validation treats the tagged value as a string because its return type is `string`.

## Clients

This repository only contains the server implementation. Here are some known clients consuming this server:

- [Eclipse Che](https://www.eclipse.org/che/)
- [vscode-yaml](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) for VS Code
- [coc-yaml](https://github.com/neoclide/coc-yaml) for [coc.nvim](https://github.com/neoclide/coc.nvim)
- [Eclipse Wild Web Developer](https://marketplace.eclipse.org/content/eclipse-wild-web-developer-web-development-eclipse-ide) for Eclipse IDE
- [lsp-mode](https://github.com/emacs-lsp/lsp-mode) for Emacs
- [vim-lsp](https://github.com/prabirshrestha/vim-lsp) for Vim
- [LSP-yaml](https://packagecontrol.io/packages/LSP-yaml) for Sublime Text
- [monaco-yaml](https://monaco-yaml.js.org) for Monaco editor
- [Vim-EasyComplete](https://github.com/jayli/vim-easycomplete) for Vim/NeoVim
- [nova-yaml](https://github.com/robb-j/nova-yaml/) for Nova
- [volar-service-yaml](https://github.com/volarjs/services/tree/master/packages/yaml) for Volar
- [Kate](https://kate-editor.org/)
- [yaml-schema-lint](https://github.com/X-Guardian/yaml-schema-lint), a CLI for schema linting YAML files

## Integrating

yaml-language-server can be integrated with editors, IDEs, CLIs, and other tools that support the Language Server Protocol. See [Clients](#clients) for known integrations.

You can also launch the server directly and connect to it from your own LSP client.

### Using the npm Package

Install yaml-language-server globally:

```sh
npm install -g yaml-language-server
```

Start the server with the communication channel required by your client:

```sh
yaml-language-server --stdio
yaml-language-server --socket=<port>
yaml-language-server --node-ipc
```

### Using a Local Build

Clone this repository:

```sh
git clone https://github.com/redhat-developer/yaml-language-server.git
cd yaml-language-server
```

Install dependencies and build the server:
```bash
npm install
npm run build
```

The built server is located at `./out/server/src/server.js`.

Run the built server with the communication channel required by your client:

```sh
node ./out/server/src/server.js --stdio
node ./out/server/src/server.js --socket=<port>
node ./out/server/src/server.js --node-ipc
```

### Using the Container Image

The container image is published at `quay.io/redhat-developer/yaml-language-server`.

To run the server over stdio:

```sh
docker run -i --rm quay.io/redhat-developer/yaml-language-server:latest
```

To run the server on a socket:

```sh
docker run --rm -p <port>:<port> quay.io/redhat-developer/yaml-language-server:latest --socket=<port>
```

## Language Server Protocol

yaml-language-server uses `vscode-languageserver@^9.0.0`, which implements [LSP 3.17](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/).

### Custom Schema Selection Messages

yaml-language-server defines the following custom LSP messages for schema selection.

#### SupportSchemaSelection Notification

Sent from the language client to the server to indicate that the client supports schema selection.

_Notification:_

- method: `'yaml/supportSchemaSelection'`
- params: `void`

#### SchemaStoreInitialized Notification

Sent from the server to the language client after Schema Store initialization finishes. After receiving this notification, the client can request schema information from the server.

_Notification:_

- method: `'yaml/schema/store/initialized'`
- params: `void`

#### GetAllSchemas Request

Sent from the language client to the server to retrieve all known schemas. The document URI is used to mark which schemas apply to the current document.

_Request:_

- method: `'yaml/get/all/jsonSchemas'`;
- params: document URI

_Response:_

- result: `JSONSchemaDescriptionExt[]`

```typescript
interface JSONSchemaDescriptionExt {
  /**
   * Schema URI
   */
  uri: string;
  /**
   * Schema name from Schema Store
   */
  name?: string;
  /**
   * Schema description from Schema Store
   */
  description?: string;
  /**
   * Whether this schema is used for the current document
   */
  usedForCurrentFile: boolean;
  /**
   * Whether this schema comes from Schema Store
   */
  fromStore: boolean;
}
```

#### GetSchemas Request

Sent from the language client to the server to retrieve the schemas used for the current document. Clients can use this request to show which schemas are currently active for that document.

_Request:_

- method: `'yaml/get/jsonSchema'`;
- params: document URI

_Response:_

- result: `JSONSchemaDescription[]`

```typescript
interface JSONSchemaDescriptionExt {
  /**
   * Schema URI
   */
  uri: string;
  /**
   * Schema name from Schema Store
   */
  name?: string;
  /**
   * Schema description from Schema Store
   */
  description?: string;
}
```

## Development

### Prerequisites

- [Node.js](https://nodejs.org/) v18.18.0 or higher
- npm

### Setup

Fork and clone this repository, then install dependencies:

```sh
cd yaml-language-server
npm install
```

### Build

Build the language server:
```sh
npm run build
```

The main server output is generated in `out/server/src`.

Use `npm test` to run tests.

### Module Builds

Building YAML Language Server produces [CommonJS](http://www.commonjs.org/) output in the `out/server/src` directory. In addition, a build also produces [UMD](https://github.com/umdjs/umd) (Universal Module Definition) modules and [ES Modules](https://tc39.es/ecma262/#sec-modules) (ESM) in the `lib` directory. These module formats support different server-side module loaders and browser bundlers such as webpack.

### CI

GitHub Actions publish each change in the `main` branch to the [npm registry](https://www.npmjs.com/package/yaml-language-server) with the `next` tag.
You may use the `next` version to adopt the latest changes into your project.
