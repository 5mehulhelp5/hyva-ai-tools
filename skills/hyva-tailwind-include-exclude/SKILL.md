---
name: hyva-tailwind-include-exclude
description: Add a module path to the `include` or `exclude` list in `hyva.config.json` for Tailwind CSS compilation. Use this skill when the user wants to include a module in, or exclude a module from, Tailwind CSS compilation for a Hyvä theme. Trigger phrases include "include module", "exclude module", "add to include", "add to exclude", "hyva tailwind include", "hyva tailwind exclude".
requires: hyva-theme-list, hyva-compile-tailwind-css
---

# Add Module to Tailwind Include or Exclude List

Adds a module path as a `{ "src": "<PATH>" }` entry to either the `tailwind.include` or `tailwind.exclude` array in a theme's `hyva.config.json`.

**Skill deps** (read via `<skill_path>/../{name}/SKILL.md`): `hyva-theme-list`, `hyva-compile-tailwind-css`

## Step 1: Determine the Target List

Decide whether the user wants to **include** or **exclude** the module in Tailwind CSS compilation:

- If the user's request clearly indicates one (e.g. "exclude", "include"), use it.
- Otherwise, ask the user whether to include or exclude the module. Wait for the answer before proceeding.

The chosen list is referred to as `<target>` (`include` or `exclude`) in the steps below.

## Step 2: Find hyva.config.json

Use the `hyva-theme-list` skill to find all Hyvä themes. For each theme, the config file is located at `<theme-path>/web/tailwind/hyva.config.json`.

- If only one theme is found, use it directly.
- If multiple themes are found, list them and ask the user which theme(s) to update. Wait for the answer before proceeding.
- If a selected theme is located under `vendor/`, warn the user that changes to vendor files are lost on `composer update` and ask for confirmation before proceeding.

## Step 3: Resolve the Module Path

Use the current working directory as the project root. If a module name was provided by the user when invoking the skill, use it. Otherwise, prompt the user for the module to resolve.

- If the provided value contains `/` (e.g. `vendor/vendor-name/module-name`), verify the directory exists relative to the project root and use it as-is. If it does not exist, warn the user and ask how to proceed.
- If the provided value is a `Vendor_Module` name (e.g. `Hyva_Checkout`), look for `app/code/<Vendor>/<Module>` relative to the project root and use that path if it exists. If it is not found there, look it up under `vendor/` via its `registration.php`.
- Otherwise (a plain directory name, e.g. `magento2-hyva-checkout`), search for a directory matching the name under `vendor/` and `app/code/`:
  - If no match is found, inform the user and stop.
  - If exactly one match is found, derive its relative path from the project root (e.g. `vendor/hyva-themes/magento2-hyva-checkout`).
  - If multiple matches are found, list them and ask the user which one to use. Wait for the answer before proceeding.

## Step 4: Update Each Target File

For each target `hyva.config.json`:

1. Read the file. If it does not exist, create it with the structure `{ "tailwind": { "<target>": [] } }`. If the file exists but is not valid JSON, inform the user and stop without overwriting it.
2. Check if an entry with the resolved path already exists in `tailwind.<target>`. If it does, inform the user and skip that file. If the path exists in the opposite list (`tailwind.exclude` when including, or `tailwind.include` when excluding), warn the user and ask whether to remove it from there.
3. Add `{ "src": "<PATH>" }` to the `tailwind.<target>` array, where `<PATH>` is the resolved module path from Step 3. Create the `tailwind` key and the `<target>` array if they are missing. For example, excluding a module results in:
   ```json
   {
     "tailwind": {
       "exclude": [
         { "src": "vendor/hyva-themes/magento2-hyva-checkout" }
       ]
     }
   }
   ```
4. Write the updated JSON back, preserving the file's existing indentation and trailing newline.

## Step 5: Confirm

Report which file(s) were updated, which list (`include` or `exclude`) was changed, and what path was added.

## Step 6: Offer to Compile

The change has no visible effect until Tailwind CSS is recompiled. Offer to compile the Tailwind CSS for the updated theme(s) using the `hyva-compile-tailwind-css` skill so the change takes effect.

<!-- Copyright © Hyvä Themes https://hyva.io. All rights reserved. Licensed under OSL 3.0 -->
