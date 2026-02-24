---
name: composer-local-package
description: Switch a Composer package between its Packagist version and a local path repository (symlinked), or revert back.
---

# composer-local-package

Switch a PHP Composer package from its live Packagist version to a local symlinked copy for development, or revert back to the original version.

## When to use

Use this skill when the user wants to:
- Work on a Composer dependency locally (e.g., debug or develop a package in-place)
- Switch a package to use a local path with symlinking
- Revert a previously switched package back to its original Packagist version
- See which packages are currently linked locally

Trigger phrases: "use local package", "switch to local", "link package locally", "revert package", "unlink local package", "composer local", "composer link"

## Arguments

The only required input is the **package name** (e.g., `spatie/laravel-error-handler` or just `laravel-error-handler`).

## Detect which method to use

Before performing any link/unlink operation, check if the `composer link` plugin is available:

```bash
composer link --help 2>/dev/null
```

- If the command succeeds (exit code 0), use **Method A (composer-link plugin)**.
- If the command fails, use **Method B (manual composer.json editing)**.

---

## Method A: composer-link plugin (preferred)

This method uses the `sandersander/composer-link` Composer plugin. It creates symlinks **without modifying `composer.json` or `composer.lock`**.

### Link a package

1. **Parse the package name.** If the user provides just the package name without a vendor prefix (e.g., `laravel-error-handler`), assume the full name is `spatie/{package-name}`. If they provide a full vendor/package name, use that.

2. **Check if the local directory exists.** Look for `../{package-name}` relative to the project root.

3. **If the directory does NOT exist**, try to clone it:
   ```bash
   git clone git@github.com:{vendor}/{package-name}.git ../{package-name}
   ```
   If cloning fails, stop and ask the user where the package source lives.

4. **Link the package:**
   ```bash
   composer link ../{package-name}
   ```

5. **Verify** by running `composer linked` and confirming the package appears in the list.

### Unlink a package

1. **Unlink the package:**
   ```bash
   composer unlink ../{package-name}
   ```

2. **Verify** by running `composer linked` and confirming the package no longer appears.

### Show linked packages

Run `composer linked` to display all currently linked packages.

---

## Method B: manual composer.json editing (fallback)

Use this method only when the composer-link plugin is not installed.

### Link a package

1. **Parse the package name.** If the user provides just the package name without a vendor prefix (e.g., `laravel-error-handler`), assume the full name is `spatie/{package-name}`. If they provide a full vendor/package name, use that.

2. **Read the current `composer.json`** in the project root. Note the current version constraint for the package in `require` or `require-dev`.

3. **Check if the local directory exists.** Look for `../{package-name}` relative to the project root.

4. **If the directory does NOT exist**, try to clone it:
   ```bash
   git clone git@github.com:{vendor}/{package-name}.git ../{package-name}
   ```
   If cloning fails, stop and ask the user where the package source lives.

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

### Unlink a package

1. **Determine the original state of `composer.json` before the local switch.** Use this strategy in order:
   - First, run `git diff composer.json`. If there are uncommitted changes, parse the diff to extract the original version constraint (lines prefixed with `-`) and the original `repositories` array state.
   - If `composer.json` has no uncommitted changes, run `git show HEAD:composer.json` and parse the committed version to find the original version constraint and repositories.

2. **Modify `composer.json`:**
   - Remove the path repository entry for `../{package-name}` from the `repositories` array.
   - If the `repositories` array is now empty, remove the `repositories` key entirely.
   - Restore the original version constraint for the package in `require` or `require-dev`.

3. **Run `composer update {vendor}/{package-name}`** to switch back to the Packagist version.

4. **Verify** that `vendor/{vendor}/{package-name}` is no longer a symlink.
