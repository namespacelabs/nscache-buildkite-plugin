# NSCloud Cache Plugin for Buildkite

Mounts cache volumes for [Buildkite](https://buildkite.com/) pipelines
[running on Namespace Cloud](https://cloud.namespace.so/docs/features/on-demand-buildkite-agents).

## Example

Add the following to your `pipeline.yml`:

```yml
steps:
  - command: npm install
    plugins:
      - namespacelabs/nscache#v0.1
    agents:
      nsc-cache-tag: my-repo/my-step
```

## How it works

When running Buildkite agents on Namespace platfrom you can specify additional agent tags modifying
the configuration of the VM the agent runs on. `nsc-cache-tag` enables mounting a cache volume for
this pipeline step. All steps having this tag will share cache volume contents (best effort).

The cache tag property enables mounting of a single volume under `/cache/` path into the agent.
The `nscache` plugin links subdirectories of the cache volume into the `paths` specified in the
configuration. For example, if `paths: node_modules` the plugin will link `./node_modules` to
`/cache/nscache/node_modules`.

You don't specify `paths` the plugin will automatically detect project platform and use the cache
for directories used for caches by the platform. Currently supported:

- NodeJS (if package.json is present in the repo root): `./node_modules`, `~/.npm`, `~/.cache/yarn`, `~/.pnpm-store`.
- Go (if go.mod is present in the repo root): `~/go/pkg/mod`, `~/.cache/go-build`.

## Advanced example

```yml
steps:
  - command: |
      # Install a package using global NPM cache:
      npm i -g license

      # Read current cache content.
      if ! cat my-cache/LICENSE
      then
        # Save the license to cache if not present.
        npx license --raw MIT > my-cache/LICENSE
      fi

    plugins:
      - namespacelabs/nscache#v0.1:
          paths:
            - my-cache # relative to the working dir
            - ~/.npm # absolute path
    agents:
      nsc-cache-tag: my-repo/my-step
      nsc-cache-size: 2g
```

## Configuration

### `paths` (optional, string or list of strings)

The paths to enable caching for. The paths may be absolute, relative (to working directory) or
home-relative (`~/...`).

If not specified, automatic platform detection is used (see above).
