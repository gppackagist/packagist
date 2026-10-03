# gppackagist.github.io/packagist

Composer index for the private WordPress plugin mirrors in the `gppackagist` org. `satis.yml` builds it with Satis and publishes it to GitHub Pages.

## Use it from a site

```json
{
  "repositories": [{ "type": "composer", "url": "https://gppackagist.github.io/packagist" }]
}
```

Packages download from the private mirror repos, so the site's `auth.json` needs a fine-grained GitHub token with read access to them:

```json
{ "github-oauth": { "github.com": "<token>" } }
```

## How it updates

1. Each mirror repo runs `.github/workflows/build.yml` daily. It calls a reusable workflow from [gppackagist/github-action-update-plugins](https://github.com/gppackagist/github-action-update-plugins), pinned by commit, which asks the vendor for the latest version and, if it's new, commits, tags, and releases it.
2. When a new version is mirrored, the same workflow mints a token from the `gppackagist-satis` GitHub App and starts `satis.yml` here.
3. `satis.yml` reads every repo in [`satis.json`](./satis.json) with an App token and publishes the index to `gh-pages`.
4. [gppackagist/packagist-monitor](https://github.com/gppackagist/packagist-monitor) checks daily that builds pass and the index matches the newest tags, and alerts in Slack.

To rebuild the index by hand, run a mirror's Build workflow with `rebuild_index` checked, or run `satis.yml` here.

## Add a plugin

1. Create a private repo in the org with a `composer.json`:

    ```json
    {
      "name": "gppackagist/<plugin-slug>",
      "type": "wordpress-plugin",
      "description": "<Plugin name>",
      "homepage": "<vendor URL>"
    }
    ```

2. Copy `.github/workflows/build.yml` and `.github/dependabot.yml` from a mirror that uses the same vendor workflow (for example `gravityforms` for Gravity Forms add-ons), and change the `with:` inputs. The [update-plugins README](https://github.com/gppackagist/github-action-update-plugins#github-workflows-plugins) lists the inputs per vendor.

3. Set on the new repo:

    | Name | Kind | Value |
    | --- | --- | --- |
    | `PACKAGIST_APP_ID` | variable | the `gppackagist-satis` App ID |
    | `PACKAGIST_APP_PRIVATE_KEY` | secret | the App's private key |
    | `LICENSE_KEY` (and any other inputs the vendor workflow needs) | secret | vendor licence |

    The org is on GitHub Free, so organization secrets don't reach private repos; each repo needs its own copy.

4. Run the Build workflow once. When it has produced a tag, add the repo to [`satis.json`](./satis.json).
