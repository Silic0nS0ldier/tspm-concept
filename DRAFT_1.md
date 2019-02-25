# SourceType

A technology agonistic, semantically versioned, file-oriented dependency manager.

## Goals

- Simplified tooling for projects spanning multiple technologies.
- Improve code quality through across touched ecosystems.
- Reduce prevalence of duplicated code.
- Minimise risk of breaking changes between non-major releases.

## Module Resolution

Module resolution occurs during dependency installation through the use of path rewrites in dependencies.

Treatment of duplicate dependencies is subject to the technologies within. As an example, PHP would only allow one version of a dependency to be installed citing language contraints, while JavaScript will quite happily handle extras.

To discourage the special kind of dependency hell that is having 10+ versions of the same dependency, duplicates will prompt a warning.

### Development Dependency Offset

To help prevent vulnerable or old versions of dependencies being used in the project development dependencies will not be considered in the initial dependency graph resolution. Development dependency resolution is then conducted with the already resolved dependencies acting as semver pinned versions.

This offset will cause dependency resolution failures in projects with dependencies that are unable to handle duplicates.

## Directory Structure

The directory structure has been designed to be consistent and predictable. The below chart illustrates this.

```
.
+-- vendor
|   +-- super
|   |   +-- d (3.2.7)
|   |   |   +-- sourcetype-pub.json
|   |   |   +-- super.d.ts
|   |   |   +-- super.js
|   |   |   +-- super.less
|   |   +-- 0.2.7
|   |       +-- sourcetype-pub.json
|   |       +-- super.js
|   |       +-- super.sass
|   +-- company-brand-kit
|   |   +-- 1.1.0-rc.1
|   |       +-- sourcetype-pub.json
|   |       +-- svg
|   |       |   +-- square.svg
|   |       |   +-- banner.svg
|   |       +-- png
|   |           +-- square.png
|   |           +-- banner.png
|   +-- bin-serve
|       +-- d (playground)
|           +-- sourcetype-pub.json
|           +-- serve.exe
|           +-- serve
+-- sourcetype.json
+-- sourcetype-lock.json
```

### Dependency Depth

All dependencies are held at the same level in the directory structure to ease auditing effort and avoid path length cap issues in certain tools.

### Direct Dependency Path

Direct dependencies are uniquely marked as such with `d` to permit easy usage of dependency content in a predictable fashion. Version information via folder name is lost by doing this however there is no significant disadvantage as usage of a dependency requires that its documentation be read, which in turn requires the version to verify.

## The Playground

In semantic versioning the <kbd>0.x.x</kbd> version range is commonly used for the release of developmental code. This however does to work so well once <kbd>1.0</kbd> has been reached making effective open development tricky. Furthermore, the tagging of versions at these volitate times is counterproductive and many send the wrong message if the intent is simply to test something that requires publishing.

The solution proposed is to have a playground to which releases can be deployed carelessly without risk. A special utility that dependents can point to during development to test fixes, new feature, etc. Depending on the success of this feature, it could be extended much further and play a key part in maintaining stability of the ecosystem.

Publishing of projects depending on playgrounds will be blocked to prevent misuse.

## File Schemas

### `sourcetype.json`

The file used within projects.

```json
{
    "$schema": "https://schema.typesource.org/sourcetype/1.0.0",
    "name": "sourcetype",
    "version": "1.0.0",
    "license": "",
    "addToPath": {
        "linux": "bin/linux/",
        "linux-x64": "bin/linux-x64/",
        "win10": "bin/win10/"
    },
    "lifecycleHooks": {
        "preInstall": "",
        "postInstall": "",
        "preUpdate": "",
        "postUpdate": "",
    },
    "commands": {
        "build": "dotnet build"
    },
    "external": {
        "coolConfig": {}
    },
    "dependencies": {
        "*": {
            "cool-cli-lib": "7.54.12"
        },
        "dev": {
            "dotnetcore": "3.0.0"
        }
    }
}
```

### `sourcetype-lock.json`

The file used to ensure a measure of consistency across development environments and cache the dependency resolution result along with any transformations to be performed to paths.

```json
{
    "$schema": "https://schema.typesource.org/sourcetype-lock/1.0.0",
    "dependencies": {
        "*": {
            "cool-cli-lib": [
                {
                    "version": "7.54.12",
                    "integrity-hash": {
                        "original": "...",
                        "current": "..."
                    },
                    "dependsOn": {
                        "cool-lib": "5.0.3"
                    }
                }
            ],
            "cool-lib": [
                {
                    "version": "5.0.3",
                    "integrity-hash": {
                        "original": "...",
                        "current": "..."
                    }
                }
            ]
        },
        "dev": {
            "dotnetcore": [
                {
                    "version": "3.0.0",
                    "integrity-hash": {
                        "original": "...",
                        "current": "..."
                    }
                }
            ]
        }
    }
}
```

### `sourcetype-pub.json`

The file used for managing a dependency.

```json
{
    "$schema": "https://schema.typesource.org/sourcetype-pub/1.0.0",
    "publishedWith": "SourceType CLI 1.25.2",
    "version": "3.64.1",
    "dependencies": {
        "cool-dep": "^3.8.236",
        "cooler-dep": "^3.1.6"
    },
    "addToPath": {
        "linux": "bin/linux/",
        "macos": "bin/macos/",
        "win": "bin/win/"
    }
}
```

## Scripts

Scripts can be manually run via;
- `run` to open a terminal
- `run -r "cmd"` to execute the provided string
- `run "script-name"` to execute predefined snippets

For convenience various hooks for various in-box commands exist. To reduce the risk of rouge scripts permissions available to each hook will vary according to their implicit permissions. Some examples of implicit permissions are;
- `test` implicitly grants reading, writing, and execution permissions
- `install` implicitly grants almost nothing

There are always cases where the granted implicit permissions may not be enough for a genuine use case, so to address this permission requirements can be declared. If these permission requirements are not impilictly met, the user will be prompted to either grant them and continue or decline and stop.

### Environment

Any command executed through the package manager recieves a contextual `PATH` variable consisting exclusively of direct dependencies, and be run through a [_Hosted PowerShell Core_](https://github.com/PowerShell/PowerShell/tree/master/docs/host-powershell) instance to permit greater consistency across platforms. To ensure that dependencies are always satisfied `PATH` will change along with the context.

Given the risk of tooling mismatches global dependencies and the challenges around dealing with them, their support is outside of the scope of this initial draft.
