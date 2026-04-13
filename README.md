# yama-pkl

**Typed [PKL](https://pkl-lang.org) schema for authoring YAMA profiles.**

[YAMA](https://yamaml.org) (Yet Another Metadata Application Profile and RDF Mapping Language) is a YAML-native language for authoring metadata application profiles. This package provides a type-safe PKL authoring layer that renders to YAMAML, giving you editor autocomplete, authoring-time validation, and modular composition via `amends` and `import`.

```
profile.pkl  ──pkl eval──→  profile.yaml  ──yama package──→  RDF, SHACL, ShEx, DSP, ...
```

## Quick start

Author a profile by amending the `Profile` module:

```pkl
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/people/"

namespaces {
  ["foaf"] = "http://xmlns.com/foaf/0.1/"
  ["xsd"]  = "http://www.w3.org/2001/XMLSchema#"
}

descriptions {
  ["person"] = new Description {
    a = "foaf:Person"
    statements {
      ["name"] = new LiteralStatement {
        property = "foaf:name"
        min = 1
        max = 1
        datatype = "xsd:string"
      }
    }
  }
}
```

Render to YAMAML and build:

```sh
pkl eval -f yaml profile.pkl > profile.yaml
yama package -i profile.yaml -o dist/
```

## Why PKL?

| Concern | YAMAML (YAML) | PKL |
|---|---|---|
| Authoring-time validation | runtime only | type-checked in editor |
| Reusable base profiles | anchors/aliases (limited) | `amends` / `extends` across files |
| Editor autocomplete | generic YAML | full IntelliSense from schema |
| Computed values | — | functions, interpolation |
| Modular imports | single-file | HTTPS / package imports |
| Canonical format | `.yaml` | `.pkl` source, `.yaml` rendered |

A typo like `tyep: literal` fails at generation time in YAMAML; the PKL compiler catches it the moment you save the file.

## Schema

A single module `Profile.pkl` defines the following classes:

| Class | Purpose |
|---|---|
| `Profile` | Top-level document (amends this) |
| `Description` | A class of resources — shape, template, or record type |
| `Statement` *(abstract)* | Base class for statement templates |
| `LiteralStatement` | Literal values (text, numbers, dates, picklists, patterns) |
| `IriStatement` | URI/IRI references, optionally linked to another description |
| `BnodeStatement` | Inline blank-node structured values |
| `Facets` | XSD numeric and string facets |
| `IdMapping` | Identifier configuration for data-mapped descriptions |
| `DataMapping` | Data source mapping for RDF generation |
| `Defaults` | Inherited defaults for mapping |

See [`Profile.pkl`](Profile.pkl) for the full schema with inline documentation.

## Examples

The [`examples/`](examples/) directory contains four worked profiles:

- [`tbbt-characters.pkl`](examples/tbbt-characters.pkl) — a simple metadata profile with a structured blank-node address
- [`manga-catalog.pkl`](examples/manga-catalog.pkl) — a multi-description profile with linked shapes
- [`people.pkl`](examples/people.pkl) — a data-mapped profile showing RDF generation from CSV
- [`products.pkl`](examples/products.pkl) — rich constraints: facets, pattern, picklist, closed shape

Render any example with:

```sh
pkl eval -f yaml examples/tbbt-characters.pkl
```

## Development

Prerequisites: [PKL CLI](https://pkl-lang.org/main/current/pkl-cli/index.html)

```sh
# Clone the repo
git clone https://github.com/yamaml/yama-pkl.git
cd yama-pkl

# Validate the schema
pkl eval Profile.pkl

# Run the tests
pkl test tests/Profile.test.pkl

# Validate every example renders as YAML
for f in examples/*.pkl; do pkl eval -f yaml "$f" > /dev/null; done

# Build the release package locally
pkl project package
ls .out/yama@*/
```

## Releases

Releases are automated via [`.github/workflows/release.yaml`](.github/workflows/release.yaml). To cut a new version:

1. Update `version` in `PklProject`.
2. Commit and push to `main`.
3. Create a tag matching the package version exactly: **`yama@<version>`** (no `v` prefix — the `pkg.pkl-lang.org` redirect service requires the tag to match `<name>@<version>` literally).

   ```sh
   git tag yama@0.1.0
   git push origin yama@0.1.0
   ```

4. The workflow runs `pkl project package`, verifies the tag matches `PklProject`, and publishes a GitHub Release with the four artifacts:
   - `yama@<version>.zip` — the package archive
   - `yama@<version>.zip.sha256` — archive checksum
   - `yama@<version>` — package metadata JSON
   - `yama@<version>.sha256` — metadata checksum

Once the release is published, the package is resolvable at:

```
package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@<version>
```

via PKL's [`pkg.pkl-lang.org` redirect service](https://github.com/apple/pkl/discussions/85) — no additional hosting infrastructure required.

## Documentation

Full reference documentation lives at **https://yamaml.org/docs/yamaml/pkl**.

## License

[Creative Commons Attribution-ShareAlike 4.0 International](LICENSE) (CC BY-SA 4.0).
