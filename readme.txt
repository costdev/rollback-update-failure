# Rollback Update Failure

Plugin Name: Rollback Update Failure
Contributors: afragen, aristath, costdev, pbiron
Tags: feature plugin, update, failure
License: MIT
Requires PHP: 5.6
Requires at least: 6.2
Tested up to: 6.2
Stable Tag: 5.0.6

This is a feature plugin for testing automatic rollback of a plugin or theme update failure.

## Description

This is a feature plugin for testing automatic rollback of a plugin or theme update failure.

It is based on the [PR](https://github.com/WordPress/wordpress-develop/pull/1492) for [#51857](https://core.trac.wordpress.org/ticket/51857). Current [PR #2225](https://github.com/WordPress/wordpress-develop/pull/2225/) and [PR #3958](https://github.com/WordPress/wordpress-develop/pull/3958) for inclusion to core.

* When updating a plugin/theme, the old version of the plugin/theme gets moved to a `wp-content/temp-backup/plugins/PLUGINNAME` or `wp-content/temp-backup/themes/THEMENAME` folder. The reason we chose to **move** instead of **zip**, is because zipping/unzipping are very resources-intensive processes, and would increase the risk on low-end, shared hosts. Moving on the other hand is performed instantly and won't be a bottleneck.
* If the update fails, then the "backup" we kept in the `temp-backup` folder gets restored to its original location
* If the update succeeds, then the "backup" is deleted
* 2 new checks were added in the site-health screen:
  * Check to make sure that the rollbacks folder is writable.
  * Check there is enough disk-space available to safely perform updates.

To avoid confusion: The "temp-backup" folder will NOT be used to "roll-back" a plugin to a previous version after an update. This folder will simply contain a **transient backup** of the previous version of a plugins/themes getting updated, and as soon as the update process finishes, the folder will be empty.

## Testing

* If the `wp-content/temp-backup` folder is not writable, there should be an error in the site-health screen.
* If the server has less than 20MB available, there should be an error in the site-health screen that updates may fail.
* If the server has less than 100MB, it should be a notice that disk space is running low.
* When updating a plugin, you should be able to see the old plugin in the `wp-content/temp-backup/plugins/PLUGINNAME` folder. The same should apply for themes. Since updates sometimes run fast and we may miss the folder creation during testing, you can simulate an update failure to demonstrate. This will return early and skip deleting the backup on update-success.
* When a plugin update fails, the previous version should be restored. To test that, change the version of a plugin to a previous number, run the update, and on fail the previous version (the one where you changed the version number) should still be installed on the site. To simulate an update failure and confirm this works, you can use the snippet below:

<pre><code>
    add_filter( 'upgrader_install_package_result', function() {
        return new WP_Error( 'simulated_error', 'Simulated Error' );
    });
</code></pre>

Alternatively you can install the [Rollback Update Testing](https://gist.github.com/afragen/80b68a6c8826ab37025b05d4519bb4bf) plugin, activating it as needed.

Or use the built-in simulate failure feature. Just activate/deactivate from the `plugins.php` page action link.

## Reporting

Please submit [issues](https://github.com/afragen/rollback-update-failure/issues) and [PRs](https://github.com/afragen/rollback-update-failure/pulls) to GitHub.

Logo from a meme generator. [Original artwork](http://hyperboleandahalf.blogspot.com/2010/06/this-is-why-ill-never-be-adult.html) by Allie Brosh.

## Changelog

Please see the Github repository: [CHANGELOG.md](https://github.com/afragen/rollback-update-failure/blob/main/CHANGELOG.md).

#### 5.0.6 / 2023-04-25
* update code logic for creating `temp-backup` dir, thanks @azaozz

#### 5.0.5 / 2023-04-14
* hotfix for no autoload

#### 5.0.4 / 2023-04-14
* update tests
* update GitHub Actions
* ignore vendor directory

#### 5.0.3 / 2023-03-22
* update @since
* update using constant to check version for when `move_dir()` was committed
* update using constant to check version for when `Rollback` was committed
* update for PR compatibility
* developery stuff

#### 5.0.2 / 2023-02-05
* make variables static to retain value during auto-updater run

#### 5.0.1 / 2023-02-03
* ensure `move_dir()` called with 3rd parameter as `move_dir($from, $to, true)`

#### 5.0.0 / 2023-02-02
* during `WP_Rollback_Auto_Update::restart_updates` remove shutdown hook for `WP_Upgrader::delete_temp_backup`
* skip second sequential call to `create_backup`
* now require at least WP 6.2-beta1, deactivate if requirements not met
* Faster Updates no longer required as [committed to core](https://core.trac.wordpress.org/changeset/55204)

#### 4.1.2 / 2023-01-25
* update `move_dir()` for new parameter

#### 4.1.1 / 2023-01-20
* ensure specific functions are loaded to check for Faster Updates

#### 4.1.0 / 2023-01-19
* change directory name of rollback to distinguish from update.
* update for `move_dir()` possibly returning `WP_Error`
* fix `sprintf` error

#### 4.0.0 / 2023-01-10
* cast `upgrade_plugins` transient to object, overkill but someone reported an error
* merge Rollback Auto Update
* require [Faster Updates](https://github.com/afragen/faster-updates) for `move_dir()`, auto-install/activate
* no longer requires special filter in `WP_Upgrader::install_package`
* testing only on `update-core.php`

#### 3.3.2 / 2022-12-30
* update for [new filter hook in WP_Upgrader::install_package](https://github.com/WordPress/wordpress-develop/pull/3791)
* update nonce verification for failure simulator

#### 3.3.1 / 2022-10-25
* use `array_unique` when saving simulated failure options
* load failure simulator in `init` hook for WP-CLI

#### 3.3.0 / 2022-10-14
* use `wp-content/temp-backup` and not `wp-content/upgrade/temp-backup` as `WP_Upgrader::unpack_package` deletes contents of `wp-content-upgrade` at each update
* add simulated failure into plugin

#### 3.2.1 / 2022-09-23
* bump auto-deactivation check for WP version

#### 3.2.0 / 2022-09-19
* backup runs on `upgrader_source_selection` from `upgrader_pre_install` to resolve an edge case
* rename functions for action not hook

#### 3.1.1 / 2022-07-31
* update VirtualBox testing URL in readme(s)

#### 3.1.0 / 2022-06-27
* fix to ensure restore functions correctly during bulk update

#### 3.0.0 / 2022-06-14
* remove references to VirtualBox
* add `pre_move_dir` and `post_move_dir` hooks
* use with VirtualBox environment will require a [mu-plugin and a watcher script](https://gist.github.com/costdev/502a2ca52a440e5775e2db970227b9b3) or similar for VirtualBox based environments
* update error messaging in `delete_temp_backup()`

#### 2.2.0 / 2022-05-11
* add initial setup of weekly `wp_delete_temp_updater_backups` cron task, oops

#### 2.1.2 / 2022-05-11
* fix `shutdown` hook in `wp_delete_all_temp_backups()` for plugin namespace, not for PR

#### 2.1.1 / 2022-05-11
* update testing workflows
* fix action hook `wp_delete_temp_updater_backups` for plugin namespace, not for PR

#### 2.1.0 / 2202-04-12
* pass basename of destination to `copy_dir( $skip_list )` to avoid potential endless looping.

#### 2.0.0 / 2022-04-06
* refactor to ease PR back into core by separating out changes into respective files/classes

#### 1.5.0 / 2022-04-04
* remove anonymous callbacks
* add class `$options` for callback functions
* update `is_virtualbox()` for testing
* add testing scaffold

#### 1.4.0 / 2022-04-03
* move kill switch to WP6.1-beta1
* add non-direct filesystem rename variants to `move_dir()`
* bring into alignment with PR

#### 1.3.6 / 2022-03-31
* update credit

#### 1.3.5 / 2022-03-31
* add more Site Health info for runtime environment
* update `move_dir()`
* add `is_virtualbox()`
* remove `WP_RUNTIME_ENVIRONMENT` and `wp_get_runtime_environment()`

#### 1.3.4 / 2022-03-21
* run `restore_temp_backup()` in `shutdown` hook

#### 1.3.3 / 2022-03-18
* add `wp_get_runtime_environment()` to return value of constant `WP_RUNTIME_ENVIRONMENT`
* allowed values are obviously up for discussion
* update to most of current PR

#### 1.3.2 / 2022-02-15
* update to correspond to core patch

#### 1.3.1 / 2022-01-19
* add logo credit, Logo from a meme generator. [Original artwork](http://hyperboleandahalf.blogspot.com/2010/06/this-is-why-ill-never-be-adult.html) by Allie Brosh.
* remove `(int)` casting for `disk_free_space()`

#### 1.3.0 / 2021-01-12
* introduce `is_virtual_box()` to get whether running in VirtualBox, requires `define( 'ENV_VB', true )` or `genenv( 'WP_ENV_VB' )` evaluating to true
* skips `rename()` as VirtualBox gets borked when using `rename()`

#### 1.2.0 / 2021-12-17
* updated for more parity with planned code
* updated version check for revert
* update to use `move_dir()` instead of `$wp_filesystem->move()`

#### 1.1.3 / 2021-09-17
* update version check

#### 1.1.1 / 2021-09-07
* update check for `disk_free_space()`

#### 1.1.0 / 2021-09-01
* automatically deactivate plugin after feature committed to core, currently set to `5.9-beta1`
* check for disabled function `disk_free_space()` and degrade gracefully

#### 1.0.0 / 2021-08-30
* updated to be on par with [PR #1492](https://github.com/WordPress/wordpress-develop/pull/1492), thanks @aristah
* original zip rollback is now branch [zip-rollback](https://github.com/WordPress/rollback-update-failure/tree/zip-rollback)

#### 0.5.3 / 2021-07-01
* add @10up GitHub Actions integration for WordPress SVN

#### 0.5.2 / 2021-06-10
* exit early if `$hook_extra` is empty

#### 0.5.1 / 2021-03-15
* update error message for installation not update

#### 0.5.0 / 2021-02-10
* initial commit
* use simpler hook for `extract_rollback`
* update for `upgrader_install_package_result` filter and parameters passed
* add text domain
* update error message display
* added filter `rollback_update_testing` to simulate a failure.
* override filter if there's already a WP_Error
