# composer-local-package

An agent skill to quickly switch a Composer package between its live Packagist version and a local path repository (symlinked) for development.

## Install

```bash
npx skills add AlexVanderbist/composer-local-package-skill
```

## Usage

Ask your agent to:

- **Switch to local:** "use local laravel-error-handler"
- **Revert:** "revert laravel-error-handler to packagist"

The skill will symlink `../{package-name}` into your project via a Composer path repository. If the directory doesn't exist, it tries cloning from `spatie/{package-name}`.
