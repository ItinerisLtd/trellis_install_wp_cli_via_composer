# trellis_install_wp_cli_via_composer

[![Ansible Role](https://img.shields.io/ansible/role/45815?style=flat-square)](https://galaxy.ansible.com/ItinerisLtd/trellis_install_wp_cli_via_composer)
[![GitHub tag (latest SemVer)](https://img.shields.io/github/v/tag/itinerisltd/trellis_install_wp_cli_via_composer?style=flat-square)](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer/releases)
[![Ansible Role](https://img.shields.io/ansible/role/d/45815?style=flat-square)](https://galaxy.ansible.com/ItinerisLtd/trellis_install_wp_cli_via_composer)
[![CircleCI](https://circleci.com/gh/ItinerisLtd/trellis_install_wp_cli_via_composer.svg?style=svg)](https://circleci.com/gh/ItinerisLtd/trellis_install_wp_cli_via_composer)
[![Ansible Quality Score](https://img.shields.io/ansible/quality/45815?style=flat-square)](https://galaxy.ansible.com/ItinerisLtd/trellis_install_wp_cli_via_composer)
[![GitHub License](https://img.shields.io/github/license/itinerisltd/trellis_install_wp_cli_via_composer.svg?style=flat-square)](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer/blob/master/LICENSE)
[![Hire Itineris](https://img.shields.io/badge/hire-Itineris-ff69b4.svg?style=flat-square)](https://www.itineris.co.uk/contact/)
[![Twitter Follow @itineris_ltd](https://img.shields.io/twitter/follow/itineris_ltd?style=flat-square&color=1da1f2)](https://twitter.com/itineris_ltd)
[![Twitter Follow @TangRufus](https://img.shields.io/twitter/follow/TangRufus?style=flat-square&color=1da1f2)](https://twitter.com/tangrufus)

Install WP-CLI via composer on Trellis servers.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Goal](#goal)
  - [The Problem](#the-problem)
  - [The Solution](#the-solution)
- [Role Variables](#role-variables)
- [Requirements](#requirements)
- [Installation](#installation)
- [FAQ](#faq)
  - [How to install certain commands only instead of the whole WP-CLI bundle?](#how-to-install-certain-commands-only-instead-of-the-whole-wp-cli-bundle)
  - [What to do when composer couldn't install packages because of  conflicting version constraints?](#what-to-do-when-composer-couldnt-install-packages-because-of--conflicting-version-constraints)
  - [How to verify WP-CLI is installed via composer?](#how-to-verify-wp-cli-is-installed-via-composer)
  - [Is it idempotent or deterministic?](#is-it-idempotent-or-deterministic)
  - [It looks awesome. Where can I find more goodies like this?](#it-looks-awesome-where-can-i-find-more-goodies-like-this)
  - [Where can I give :star::star::star::star::star: reviews?](#where-can-i-give-starstarstarstarstar-reviews)
- [Testing](#testing)
- [Feedback](#feedback)
- [Security](#security)
- [Credits](#credits)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Goal

### The Problem

> [Since WP-CLI v2.0.0,] the most problematic set of the dependencies, the hard requirement on an old version of Symfony, is gone. The only Symfony component we still have (yet) is `symfony/finder`, as there’s no upper version limit for that one.
>
> -- https://make.wordpress.org/cli/2018/08/08/wp-cli-v2-0-0-release-notes/

However, the phar bundles with [WP-CLI's dependencies](https://github.com/wp-cli/wp-cli/blob/bc37f66e25b9992813b0c50b933aefbf88718e6a/composer.lock). Those dependencies' always being loaded from the phar. As a result, their versions are _locked_.

For example: WP-CLI v2.4.0 phar bundles with [`symfony/process` v2.8.5](https://github.com/wp-cli/wp-cli-bundle/blob/v2.4.0/composer.lock#L1205-L1206) (as a dependency of `symfony/finder`). Assume we have `my-awesome-command` which requires `symfony/process:5.0.0`. `$ wp package install my-awesome-command` installs `symfony/process` v5.0.0 as expected. However, `symfony/process` v2.8.5 (from the WP-CLI phar) is always used; the newer version of `symfony/process` (which required by `my-awesome-command`) is being ignored. Thus, `my-awesome-command` fails when trying to use `symfony/process`.

```sh-session
$ wp shell

wp> $reflector = new \ReflectionClass('Symfony\Component\Process\Process');
=> object(ReflectionClass)#2801 (1) {
  ["name"]=>
  string(33) "Symfony\Component\Process\Process"
}
wp> echo $reflector->getFileName(); // Note that it is loaded form the WP-CLI phar.
phar:///usr/bin/wp/vendor/symfony/process/Process.php
```

This problem affects `symfony/finder` and its dependencies.

### The Solution

[Installing WP-CLI via composer](https://make.wordpress.org/cli/handbook/installing/#installing-via-composer) resolves the problem.

```sh-session
$ wp shell

wp> $reflector = new \ReflectionClass('Symfony\Component\Process\Process');
=> object(ReflectionClass)#2801 (1) {
  ["name"]=>
  string(33) "Symfony\Component\Process\Process"
}
wp> echo $reflector->getFileName(); // Note that it is loaded form composer's vendor folder.
/home/web/.composer/vendor/symfony/process/Process.php
```

## Role Variables

```yaml
# Composer packages to be removed before installing WP-CLI
# Default: []
wp_cli_composer_global_remove_packages:
  - wp-cli/wp-cli-bundle
  - psy/psysh

# Composer packages to be installed
# Default: "wp-cli/wp-cli-bundle:{{ wp_cli_version }}"
wp_cli_composer_global_require_packages:
  - "wp-cli/wp-cli:2.4.0"
  - "wp-cli/package-command:^2"
  - "psy/psysh:^0.9.12"
  - "xxx/yyy:'^1.2.3 || ^2.2.3'"

# WP-CLI package to be installed
# Default: []
# Taken form https://github.com/roots/trellis/blob/4425669bab0665f0c9aed92c80eb9b8c54f63e85/roles/wp-cli/defaults/main.yml#L10
wp_cli_packages:
  - "typisttech/image-optimize-command:@stable"
  - "git@github.com:Yoast/wp-cli-faker.git"

# WP-CLI path
# Default: /usr/bin/wp
# Taken form https://github.com/roots/trellis/blob/4425669bab0665f0c9aed92c80eb9b8c54f63e85/roles/wp-cli/defaults/main.yml#L3
wp_cli_bin_path: /usr/bin/wp

# WP-CLI bash completion path
# Default: /etc/bash_completion.d/wp-completion.bash
# Taken form https://github.com/roots/trellis/blob/4425669bab0665f0c9aed92c80eb9b8c54f63e85/roles/wp-cli/defaults/main.yml#L9
wp_cli_completion_path: /etc/bash_completion.d/wp-completion.bash
```

## Requirements

- [Trellis](https://github.com/roots/trellis) v1.3.0 or later
- Ansible v2.7.0 or later
- Python v3.7.6 or later

## Installation

1. Add `itinerisltd.trellis_install_wp_cli_via_composer` to `galaxy.yml`
    ```diff
      # galaxy.yml

    + - src: itinerisltd.trellis_install_wp_cli_via_composer
    + version: 0.1.1 # Check for latest version!
    ```

2. Replace `wp-cli` role with `itinerisltd.trellis_install_wp_cli_via_composer`
    ```diff
      # server.yml

    - - { role: wp-cli, tags: [wp-cli] }
    + - { role: itinerisltd.trellis_install_wp_cli_via_composer, tags: [wp-cli] }
    ```

3. Install galaxy roles
    ```bash
    trellis galaxy install
    # Alternatively
    ansible-galaxy install -r galaxy.yml --force
    ```

4. Re-provision
    ```bash
    trellis provision production
    # Alternatively
    ansible-playbook server.yml -e env=production
    ```

## FAQ

### How to install certain commands only instead of the whole WP-CLI bundle?

By default, the whole WP-CLI bundle ([`wp-cli/wp-cli-bundle`](https://packagist.org/packages/wp-cli/wp-cli-bundle)) installed. If you want to keep your servers _lean_, install command packages selectively:

```yaml
wp_cli_composer_global_remove_packages:
  - wp-cli/wp-cli-bundle

wp_cli_composer_global_require_packages:
  # Required: WP-CLI framework
  - "wp-cli/wp-cli:^2.4"
  # Only install commands you need:
  - "wp-cli/core-command:^2"
  - "wp-cli/cron-command:^2"
  - "wp-cli/db-command:^2"
  - "wp-cli/package-command:^2"
```

### What to do when composer couldn't install packages because of  conflicting version constraints?

Double check existing composer packages. Then, change [version constraints](https://getcomposer.org/doc/articles/versions.md) via [role variables](#role-variables). You might need to uninstall some packages.

These commands are your friends:

```bash
composer global show
cat $(composer config --global --absolute home)/composer.json

wp package list
cat $(wp package path)/composer.json
```

### How to verify WP-CLI is installed via composer?

```sh-session
# Bad: Installed via Trellis, i.e: the phar
$ wp cli info
WP-CLI root dir:	phar://wp-cli.phar/vendor/wp-cli/wp-cli
```

```sh-session
# Good: Installed via this role, i.e: composer
$ wp cli info
WP-CLI root dir:	/home/web/.composer/vendor/wp-cli/wp-cli
```

### Is it idempotent or deterministic?

No.

Specific exact package versions might help but you will need to manage them manually.

### It looks awesome. Where can I find more goodies like this?

- Articles on [Itineris' blog](https://www.itineris.co.uk/blog/)
- More projects on [Itineris' GitHub profile](https://github.com/itinerisltd)
- More plugins on [Itineris](https://profiles.wordpress.org/itinerisltd/#content-plugins) and [TangRufus](https://profiles.wordpress.org/tangrufus/#content-plugins) wp.org profiles
- Follow [@itineris_ltd](https://twitter.com/itineris_ltd) and [@TangRufus](https://twitter.com/tangrufus) on Twitter
- Hire [Itineris](https://www.itineris.co.uk/services/) to build your next awesome site

### Where can I give :star::star::star::star::star: reviews?

Thanks! Glad you like it. It's important to let my boss knows somebody is using this project. Since this is not hosted on wordpress.org, please consider:

- tweet something good with mentioning [@itineris_ltd](https://twitter.com/itineris_ltd) and [@TangRufus](https://twitter.com/tangrufus)
- :star: star this [Github repo](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer)
- :eyes: [watch](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer/subscription) this Github repo
- write blog posts
- submit [pull requests](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer)
- [hire Itineris](https://www.itineris.co.uk/services/)

## Testing

```bash
ansible-playbook -i 'localhost,' --syntax-check tests/test.yml
```

## Feedback

**Please provide feedback!** We want to make this library useful in as many projects as possible.
Please submit an [issue](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer/issues/new) and point out what you do and don't like, or fork the project and make suggestions.
**No issue is too small.**

## Security

If you discover any security related issues, please email [dev@itineris.co.uk](mailto:dev@itineris.co.uk) instead of using the issue tracker.

## Credits

[trellis_install_wp_cli_via_composer](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer) is a [Itineris Limited](https://www.itineris.co.uk/) project created by [Tang Rufus](https://typist.tech).

Full list of contributors can be found [here](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer/graphs/contributors).

## License

[trellis_install_wp_cli_via_composer](https://github.com/ItinerisLtd/trellis_install_wp_cli_via_composer) is released under the [MIT License](https://opensource.org/licenses/MIT).
