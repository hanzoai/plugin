# runtime

One plugin runtime for the Hanzo estate: a module declares what it exports, and
a caller invokes an export by name with JSON in and JSON out. What language the
module is written in is a property of the module, not of the caller.

```go
type Module interface {
	Name() string
	Runtime() string
	Exports() []string
	Invoke(ctx context.Context, fn string, payload []byte) ([]byte, error)
}
```

That interface is `zip`'s. This module implements it rather than declaring its
own copy, which is the whole reason it exists: the same contract was written
down twice — once in zip, once beside the loader — and only structural typing
kept the two honest.

## The runtimes

| runtime      | language                  | how |
|--------------|---------------------------|-----|
| `native`     | Go                        | in-process, no boundary |
| `goja`       | JavaScript, TypeScript    | in-process, no Node |
| `wazero`     | Rust, and anything else   | compiled to WebAssembly ahead of time |
| `pyvm`       | Python                    | CPython via cgo, PEP 684 sub-interpreters — build tag `pyvm` |

`wazero` is why this list is not a list of four languages. Anything that
compiles to WebAssembly arrives without a new runner, and Rust is the first
caller of that door rather than a special case built for it.

## A module declares itself

`extension.json`, beside the module:

```json
{
  "name": "brave-search",
  "version": "1.0.0",
  "runtime": "goja",
  "module": "index.js",
  "exports": ["search"]
}
```

`Exports` is the contract. A caller reads it to know what can be invoked, and
nothing else has to be registered anywhere — which is what lets one registry
publish a Go action and a Python one side by side.
