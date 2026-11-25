# layered-actions-github

The GitHub Actions workflows every [Layered](https://github.com/layered-ai-public) app runs, kept in one place so a change here reaches every app.

Each app, generated from [layered-foundation-rails](https://github.com/layered-ai-public/layered-foundation-rails), has a short file of the same name in its own `.github/workflows` that uses one of these. The app's file keeps what GitHub needs in the app's repository: the triggers, the run's name (which is how Layered finds a run it started), permissions, concurrency, and the secrets passed on. The steps are here.

Most are [reusable workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows), in `.github/workflows`. The three that work on a server are [composite actions](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-composite-action), each in a directory of its own, for the reason under [Secrets](#secrets).

| Workflow or action | Runs |
| --- | --- |
| `ci.yml` | Brakeman, bundler-audit, importmap audit, RuboCop, tests and system tests |
| `setup.yml` | The one-time setup task that renames a new app, on a branch with a pull request |
| `deploy` | `bin/kamal setup` in a GitHub Environment |
| `remove` | Takes an environment off its server |
| `server-keys` | Puts a project's deploy keys on a server another environment already deploys to |
| `claude.yml` | Claude Code, on an `@claude` mention |
| `codex.yml` | Codex, on an issue mentioning `@codex` |
| `opencode.yml` | opencode, on a `/opencode` mention |

## An app's own setup

The coding agents' workflows get a checkout ready for an app made from layered-foundation-rails: `libvips`, Ruby and the app's gems. An app that needs more says so with two files in its own repository, both optional:

- **`.layered/packages`** lists apt packages, one a line, with `#` starting a comment. They're installed before the gems are, for a gem that builds against one (`libmagickwand-dev` for `rmagick`, say).
- **`.layered/setup`** is a script, run with `bash` once the gems are in, as the runner's user, which has `sudo`. It gets the checkout ready for the agent to run the app and its tests: starting a database, creating it, loading its schema. A line it appends to `$GITHUB_ENV` is set for the agent:

  ```sh
  sudo systemctl start postgresql
  echo "RAILS_ENV=test" >> "$GITHUB_ENV"
  bin/rails db:prepare
  ```

Both are read from the branch the agent starts from. The script is given none of the workflow's secrets, and a failure in it fails the run before the agent starts.

## Secrets

The coding agents' workflows take the app's repository secrets by name, as `secrets: inherit` only works between repositories with the same owner, and apps aren't owned by this repository's. A new secret has to be declared here and passed by each app's file.

`deploy`, `remove` and `server-keys` need the secrets of the GitHub Environment being deployed, which Layered writes there. GitHub gives those only to a job in the app's own repository that names the environment: a reusable workflow owned by someone else gets the environment's variables but none of its secrets, and the app's file can't pass them, since a job that calls a workflow can't name an environment. So these three are composite actions. The job stays in the app's file, naming the environment, and passes `SSH_PRIVATE_KEY` and `RAILS_MASTER_KEY` to the action as inputs.

Every app trusts this repository with its deploy keys, its Rails master key and its agents' API keys. Keep write access to it, and to its tags, to as few people as can manage it.

## Versions

Apps call the `v1` tag. A change that works with the apps' files as they are is released by moving the tag:

```sh
git tag -f v1 && git push -f origin v1
```

A change that needs the apps' files changed too (a new trigger, input, secret or permission) is released as `v2`, with the matching change to layered-foundation-rails. Existing apps keep running `v1` until each changes `@v1` to `@v2`.

## License

Apache 2.0. See [LICENSE](LICENSE).
