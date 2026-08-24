# muon/api-schema-export

Metapackage. Installs the two halves of Muon API Schema Export together:

| Package | Repository | What it is |
|---|---|---|
| `muon/module-api-schema-export` | `magento2-module-api-schema-export` | The generator, its service contracts, and the `bin/magento` command |
| `muon/module-api-schema-export-admin-ui` | `magento2-module-api-schema-export-admin-ui` | The **System → API Schema Export** screen and its downloads |

The core module is genuinely usable alone — it carries the whole domain and a console command, and
deliberately does not depend on `Magento_Backend`, which is what keeps it installable in a headless
or CI context. The admin module is the one that cannot stand by itself: it is a front-end over the
core module's `ExportManagerInterface` and references an ACL resource the core module declares.

Most installations want both, and want their versions moving together. That is what this package
expresses.

Nothing is installed by this package itself: `type: metapackage` means Composer resolves it purely as
a set of requirements.

## What it does

Magento declares its REST surface across dozens of per-module `etc/webapi.xml` files and then merges
them, discarding which module declared what. The built-in `/rest/all/schema` endpoint therefore
serves one whole-installation document with no attribution. These modules restore the attribution and
turn any selection of modules into a `.http` file, a Postman v2.1 collection, an OpenAPI 3.1 document
or a Swagger 2.0 document.

## Installing

These packages are not on Packagist, so Composer needs to be told where to find them. The
repositories are public, so no authentication is required. Version constraints resolve from git
tags, which is why each repository is tagged.

```bash
composer config repositories.muon-api-schema-export vcs https://github.com/muon-m2/magento2-module-api-schema-export
composer config repositories.muon-api-schema-export-admin-ui vcs https://github.com/muon-m2/magento2-module-api-schema-export-admin-ui
composer config repositories.muon-api-schema-export-meta vcs https://github.com/muon-m2/magento2-api-schema-export

composer require muon/api-schema-export:^1.0
```

All three entries are required. A metapackage lists what it needs but not where those packages live,
so adding only this repository leaves Composer unable to resolve either dependency.

### Core module only

If you want the CLI without the admin screen — a documentation build server, for instance — require
the core module directly rather than this metapackage:

```bash
composer require muon/module-api-schema-export:^1.0
```

## After installing

```bash
bin/magento module:enable Muon_ApiSchemaExport Muon_ApiSchemaExportAdminUi
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento cache:flush

bin/magento muon:api-schema:export Muon --format=all
```

Neither module has a declarative schema, so no whitelist generation is needed and `setup:upgrade`
only registers them.

The admin screen appears under **System → API Schema Export** for roles holding
*Generate API Schema* (`Muon_ApiSchemaExport::export`).

## Magento and PHP compatibility

Verified against the real package sets for each release:

| | Magento 2.4.7 | Magento 2.4.8 | Magento 2.4.9 |
|---|---|---|---|
| `magento/framework` | 103.0.7 | 103.0.8 | 103.0.9 |
| `magento/module-webapi` | 100.4.6 | 100.4.7 | 100.4.8 |
| `magento/module-backend` | 102.0.7 | 102.0.8 | 102.0.9 |
| `magento/module-store` | 101.1.7 | 101.1.8 | 101.1.9 |
| PHP | 8.1 – 8.3 | 8.2 – 8.4 | 8.3 – 8.5 |
| **Supported** | **yes** | **yes** | **yes** (tested here) |

Every framework API these modules use was checked to exist at each release tag, including the two
behaviours they actually depend on rather than merely reference: `FileFactory::create()` honouring
`rm` only for the array content form, and `Magento_WebapiAsync` deriving its variants from every
non-GET route.

PHP floor is **8.1**, confirmed with PHPCompatibility rather than assumed — the code uses readonly
promoted properties and nothing newer.

### One caveat on 2.4.7

Magento 2.4.7 ships **PHPUnit 9.5**, which has no support for PHP attributes. The modules' unit tests
declare their data providers with both the `#[DataProvider]` attribute and the `@dataProvider`
annotation so the suite runs unchanged on PHPUnit 9.5, 10.5 and 12. This affects only the shipped
tests; production code is unaffected either way.

## Versioning

Both packages are versioned together and released as a pair. `^1.0` on each is deliberate: a minor
release of one is expected to work with the matching minor of the other.

## Documentation

The core module repository carries the documentation set — a
[technical reference](https://github.com/muon-m2/magento2-module-api-schema-export/blob/main/docs/technical-reference.md)
and a
[developer guide](https://github.com/muon-m2/magento2-module-api-schema-export/blob/main/docs/developer-guide.md)
covering the two extension points: the renderer pool, and the
`muon_api_schema_export_surface_resolved` event. The admin module repository carries the
[user guide](https://github.com/muon-m2/magento2-module-api-schema-export-admin-ui/blob/main/docs/user-guide.md).

## License

OSL-3.0 — see [LICENSE.txt](LICENSE.txt). Both bundled modules carry the same licence.
