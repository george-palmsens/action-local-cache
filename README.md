# Local Cache Github Action

This github action allows you to save and restore files across job runs directly in the runner's file system.

It is intended to be used with runners with persisted storage. Don't use this action if you're using Github-hosted runners or self-hosted runners in ephemeral instances (like docker containers that are launched on demand when a workflow starts and are terminated when the job finishes).


## Usage

```yaml
# .github/workflows/my-workflow.yml
jobs:
  my_job:

    steps:
      - uses: actions/checkout@v2

      - name: Local cache for API dependencies
        id: api-cache
        uses: MasterworksIO/action-local-cache@main
        with:
          path: './api/node_modules/'
          key: 'api-dependencies-v1'

      - name: Install dependencies
        run: cd api/ && npm install
        if: steps.api-cache.outputs.cache-hit != 'true'

      - name: Do your stuff
        run: npm run build
```

The `key` param is optional and you can use it to invalidate cache.

# How it works

The first time `action-local-cache` is used in a runner it will take the given path and create a folder structure inside the `$RUNNER_TOOL_CACHE` dir (usually `_work/_tool` inside the runner's workspace), prefixing the user/org name, repo name, and key.

For example, when running the usage example above inside a workflow for the `MasterworksIO/product` repo, this empty folder will be created:

```shell
~/runner/_work/_tool/MasterworksIO/product/api-dependencies-v1/
```

Since there's no `api/node_modules/` dir inside, the restore step is skipped and the `cache-hit` output variable will be set to `false`; because of that, the next step which install the dependencies will run.

After the job is successfully completed, `action-local-cache` will take the `api/node_modules` dir generated by the install dependencies step and move it to the previously created dir structure before the runner workspace is wiped.

On a next run under the same runner instance, the cache will be moved back to the working directory and the `cache-output` variable will be set to `true`.

After all steps complete the folder will be moved back to the cache dir and so on. Since it is using the filesystem move syscalls, cache saving and restoration is instant.

# LICENSE

MIT, see [LICENSE](./LICENSE)
