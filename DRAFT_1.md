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

### `sourcetype-lock.json`

### `sourcetype-pub.json`
