# layered-actions-github

The GitHub Actions workflows every [Layered](https://github.com/layered-ai-public) app runs, kept in one place so a change here reaches every app.

Each app, generated from [layered-foundation-rails](https://github.com/layered-ai-public/layered-foundation-rails), has a short file of the same name in its own `.github/workflows` that calls one of these as a [reusable workflow](https://docs.github.com/en/actions/sharing-automations/reusing-workflows). The app's file keeps what GitHub needs in the app's repository: the triggers, the run's name (which is how Layered finds a run it started), permissions, concurrency, and the secrets passed on. The steps are here.

| Workflow | Runs |
| --- | --- |
| `ci.yml` | Brakeman, bundler-audit, importmap audit, RuboCop, tests and system tests |
| `setup.yml` | The one-time setup task that renames a new app, on a branch with a pull request |
| `deploy.yml` | `bin/kamal setup` in a GitHub Environment |
| `remove.yml` | Takes an environment off its server |
| `server-keys.yml` | Puts a project's deploy keys on a server another environment already deploys to |
| `claude.yml` | Claude Code, on an `@claude` mention |
| `codex.yml` | Codex, on an issue mentioning `@codex` |
| `opencode.yml` | opencode, on a `/opencode` mention |

## Secrets

`deploy.yml`, `remove.yml` and `server-keys.yml` read the secrets and variables of the environment they run in, in the app's repository, so nothing is passed to them. The coding agents' workflows take the app's repository secrets by name, as `secrets: inherit` only works between repositories with the same owner, and apps aren't owned by this repository's. A new secret has to be declared here and passed by each app's file.

Every app trusts this repository with its deploy keys, its Rails master key and its agents' API keys. Keep write access to it, and to its tags, to as few people as can manage it.

## Versions

Apps call the `v1` tag. A change that works with the apps' files as they are is released by moving the tag:

```sh
git tag -f v1 && git push -f origin v1
```

A change that needs the apps' files changed too (a new trigger, input, secret or permission) is released as `v2`, with the matching change to layered-foundation-rails. Existing apps keep running `v1` until each changes `@v1` to `@v2`.

## License

Apache 2.0. See [LICENSE](LICENSE).
