---
name: composer-local-package
description: Switch a Composer package between its Packagist version and a local path repository (symlinked), or revert back.
---

# composer-local-package

Switch a PHP Composer package from its live Packagist version to a local path repository (symlinked) for development, or revert back to the original version.

## When to use

Use this skill when the user wants to:
- Work on a Composer dependency locally (e.g., debug or develop a package in-place)
- Switch a package to use a local path repository with symlinking
- Revert a previously switched package back to its original Packagist version

Trigger phrases: "use local package", "switch to local", "link package locally", "revert package", "unlink local package", "composer local"

## Arguments

The only required input is the **package name** (e.g., `spatie/laravel-error-handler` or just `laravel-error-handler`).

## Instructions: Switch to local path repository

1. **Parse the package name.** If the user provides just the package name without a vendor prefix (e.g., `laravel-error-handler`), assume the full name is `spatie/{package-name}`. If they provide a full vendor/package name, use that.

2. **Read the current `composer.json`** in the project root. Note the current version constraint for the package in `require` or `require-dev`.

3. **Check if the local directory exists.** Look for `../{package-name}` relative to the project root (e.g., if the package is `spatie/laravel-error-handler`, check for `../laravel-error-handler`).

4. **If the directory does NOT exist**, try to clone it:
   ```bash
   git clone git@github.com:spatie/{package-name}.git ../{package-name}
   ```
   - If cloning fails (repo doesn't exist), stop and ask the user where the package source lives.

5. **Modify `composer.json`:**
   - Add a path repository entry to the `repositories` array (create the array if it doesn't exist). Place it at the beginning of the array:
     ```json
     {
       "type": "path",
       "url": "../{package-name}"
     }
     ```
   - Change the package version constraint to `"*"` in `require` or `require-dev` (whichever section it's currently in).

6. **Run `composer update {vendor}/{package-name}`** to install the local symlinked version.

7. **Verify** the symlink was created by checking that `vendor/{vendor}/{package-name}` is a symlink.

## Instructions: Revert to original Packagist version

1. **Determine the original state of `composer.json` before the local switch.** Use this strategy in order:
   - First, run `git diff composer.json`. If there are uncommitted changes, parse the diff to extract the original version constraint (lines prefixed with `-`) and the original `repositories` array state.
   - If `composer.json` has no uncommitted changes, run `git show HEAD:composer.json` and parse the committed version to find the original version constraint and repositories.

2. **Modify `composer.json`:**
   - Remove the path repository entry for `../{package-name}` from the `repositories` array.
   - If the `repositories` array is now empty, remove the `repositories` key entirely.
   - Restore the original version constraint for the package in `require` or `require-dev`.

3. **Run `composer update {vendor}/{package-name}`** to switch back to the Packagist version.

4. **Verify** that `vendor/{vendor}/{package-name}` is no longer a symlink.
