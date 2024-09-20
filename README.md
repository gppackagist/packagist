# gppackagist.github.io/packagist

### Add a plugin

1. Create repository with a basic `composer.json`

    ```json
    {
      "name": "gppackagist/<plugin-slug>",
      "type": "wordpress-plugin",
      "description": "Plugin name",
      "homepage": "https://woocommerce.com/products/..."
    }
    ```

2. Add `.github/workflows/build.yml` which checks for plugin updates, makes releases and triggers packagist update ([`gppackagist/github-action-update-plugins` README has morere examples](https://github.com/gppackagist/github-action-update-plugins?tab=readme-ov-file#github-workflows-plugins)).

    ```yml
    name: Build
    on:
      workflow_dispatch:
      schedule:
        - cron: '5 4 * * *'
    jobs:
      build:
        uses: gppackagist/github-action-update-plugins/.github/workflows/wccom-update.yml@
        secrets:
          ACCESS_TOKEN: ${{ secrets.WCCOM_ACCESS_TOKEN }}
          ACCESS_TOKEN_SECRET: ${{ secrets.WCCOM_ACCESS_TOKEN_SECRET }}
        with:
          slug: 'woocommerce-subscriptions'
          changelog_extract: "'/[0-9\\-]+ - version/ { if (p) { exit }; if ($4 == ver) { p=1; next } } p && NF' changelog.txt"
      update-satis:
        needs: build
        if: needs.build.outputs.updated == 'true'
        uses: gppackagist/packagist/.github/workflows/update.yml@
        secrets:
          token: ${{ secrets.PACKAGIST_UPDATE_PAT }}
    ```

3. Add [`@gppackagist/deploy`](https://github.com/orgs/gppackagist/teams/deploy) as a collaborator with `read` access. **Do _NOT_ grant write access.**

4. Change the [`PACKAGIST_UPDATE_PAT`](https://github.com/organizations/gppackagist/settings/secrets/actions/PACKAGIST_UPDATE_PAT) secret permissions to be allowed to be used by the repository.

5. Add the plugin to [`satis.json`](./satis.json) in this repository.

6. Trigger an initial build which will download the latest plugin version, commit it, tag it, push it, release it and if successful trigger a rebuild in this repository.

7. Whenever a new version is found using the cron schedule, the plugin will be updated, released and finally a rebuild of this repository will be once again triggered.

### Prerequisites

- [`gppackagist_DEPLOY_PAT`](https://github.com/organizations/gppackagist/settings/secrets/actions/gppackagist_DEPLOY_PAT) action secret containg a Personal Access Token of `@gppackagist/deploy` with Conents (read) access. This is used by this repo to read plugin tags/contents from every package listed in `satis.json`. [(settings link)](https://github.com/organizations/gppackagist/settings/personal-access-tokens/367034)
- [`PACKAGIST_UPDATE_PAT`](https://github.com/organizations/gppackagist/settings/secrets/actions/PACKAGIST_UPDATE_PAT) action secret containg a Personal Access Token of a user with _write_ access to this repository. The token is limited to only this repository with Contents (write) access. [(settings link)](https://github.com/organizations/gppackagist/settings/personal-access-tokens/367065)
- [`WCCOM_ACCESS_TOKEN`](https://github.com/organizations/gppackagist/settings/secrets/actions/WCCOM_ACCESS_TOKEN) and [`WCCOM_ACCESS_TOKEN_SECRET`](https://github.com/organizations/gppackagist/settings/secrets/actions/WCCOM_ACCESS_TOKEN_SECRET) action secrets which contain the OAuth2 tokens stored in wp_options of the store which is connected to WooCommerce.
- `LICENSE_KEY` is added as a Repository Secret to each relevant plugin repository. The secret contains the license key.
