# Publish Extensions to Open VSX

[![Gitpod Ready-to-Code](https://img.shields.io/badge/Gitpod-ready--to--code-908a85?logo=gitpod)](https://gitpod.io/#https://github.com/open-vsx/publish-extensions)
[![GitHub Workflow Status](https://github.com/open-vsx/publish-extensions/workflows/Publish%20extensions%20to%20open-vsx.org/badge.svg)](https://github.com/open-vsx/publish-extensions/actions?query=workflow%3A%22Publish+extensions+to+open-vsx.org%22)

A CI script for publishing open-source VS Code extensions to [open-vsx.org](https://open-vsx.org).

## When to Add an Extension?

A goal of Open VSX is to have extension maintainers publish their extensions [according to the documentation](https://github.com/eclipse/openvsx/wiki/Publishing-Extensions). The first step we recommend is to open an issue with the extension owner. If the extension owner is unresponsive for some time, this repo (publish-extensions) can be used **as a temporary workaround** to esure the extension is published to Open VSX.

In the long-run it is better for extension owners to publish their own plugins because:

1. Any future issues (features/bugs) with any published extensions in Open VSX will be directed to their original repo/source-control, and not confused with this repo `publish-extensions`.
1. Extensions published by official authors are shown within the Open VSX marketplace as such. Whereas extensions published via publish-extensions display a warning that the publisher (this repository) is not the official author.
1. Extension owners who publish their own extensions get greater flexibility on the publishing/release process, therefore ensure more accuracy/stability. For instance, in some cases publish-extensions has build steps within this repository, which can cause some uploaded plugin versions to break (e.g. if a plugin build step changes).

⚠️ We accept only extensions with [OSI-approved open source licenses](https://opensource.org/licenses) here. If you want to have an extension with a proprietary or non-approved license, please ask its maintainers to publish it.

## How to Add an Extension?

To automatically publish an extension to Open VSX, simply add it to [`extensions.json`](./extensions.json) with the [options described below](#publishing-options). See [Publishing Options](#publishing-options) for a quick guide.

⚠️ Some extensions require additional build steps, and failing to execute them may lead to a broken extension published to Open VSX. Please check the extension's `scripts` section in the package.json file to find such steps; usually they are named `build` or similar. In case the build steps are included in the [vscode:prepublish](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#prepublish-step) script, they are executed automatically, so it's not necessary to mention them explicitly. Otherwise, please include them in the `prepublish` value, e.g. `"prepublish": "npm run build"`.

Click the button below to start a [Gitpod](https://gitpod.io) workspace where you can run the scripts contained in this repository:

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/open-vsx/publish-extensions)

## Publishing Options

The best way to add an extension here is to [open this repository in Gitpod](https://gitpod.io/#https://github.com/open-vsx/publish-extensions) and add a new entry to `extensions.json`. To test run:
```
EXTENSIONS=rebornix.ruby SKIP_PUBLISH=true node publish-extensions.js
```

Notes:
- Simply replace `$REPOSITORY_URL` with the extension's actual repository URL

```jsonc
    // Unique Open VSX extension ID in the form "<publisher>.<name>"
    "rebornix.ruby": {
      // Repository URL to clone and publish from. If the extension publishes `.vsix` files as release artifacts, this will determine the repo to fetch the releases from.
      "repository": "https://github.com/redhat-developer/vscode-yaml"
    },
```

## How do extensions get updated?

The publishing job auto infers the latest version published to MS marketplace using vsce and then tries to resolve vsix file using GitHub releases or a commit to build associated with a version using tags and commits around the last updated date.

## How are Extensions Published?

Every night at [03:03 UTC](https://github.com/open-vsx/publish-extensions/blob/e70fb554a5c265e53f44605dbd826270b860694b/.github/workflows/publish-extensions.yml#L3-L6), a [GitHub workflow](https://github.com/open-vsx/publish-extensions/blob/e70fb554a5c265e53f44605dbd826270b860694b/.github/workflows/publish-extensions.yml#L9-L21) goes through all entries in [`extensions.json`](./extensions.json), and checks if it needs to be published to https://open-vsx.org or not.

The [publishing process](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L58-L87) can be summarized like this:

1. [`git clone "repository"`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L61)
2. _([`git checkout "checkout"`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L63) if a `"checkout"` value is specified)_
3. [`npm install`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L68) (or `yarn install` if a `yarn.lock` file is detected in the repository)
4. _([`"prepublish"`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L70))_
5. _([`ovsx create-namespace "publisher"`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L75) if it doesn't already exist)_
6. [`ovsx publish`](https://github.com/open-vsx/publish-extensions/blob/d2df425a84093023f4ee164592f2491c32166297/publish-extensions.js#L86) (with `--yarn` if a `yarn.lock` file was detected earlier)

See all `ovsx` CLI options [here](https://github.com/eclipse/openvsx/blob/master/cli/README.md).

[publish-extensions-job]: https://github.com/open-vsx/publish-extensions/blob/master/.github/workflows/publish-extensions.yml
