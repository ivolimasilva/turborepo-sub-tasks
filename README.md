# `Turborepo` package sub tasks

Example repository to showcase how to use Turborepo to cache tasks that are specific to a given package.

Package `ui` has two linting tasks:
- `lint:ts`, for TypeScript files
- `lint:css`, for CSS files

However, we don't want to leak that to the root `package.json` since it's an implementation detail for that package (e.g. we don't need to expose that the `ui` package uses TypeScript or CSS files).

Still, we would like to have the benefits of Turborepo's caching mechanism without exposing the implementation details to the root `package.json`.

## Solution

Using Turborepo's [**Package configurations**](https://turborepo.com/docs/reference/package-configurations), we can define a `packages/ui/turbo.json` file that contains the specific tasks for the `ui` package.

In this case, we start by defining the _normal_ task, `lint`. This task is present in every package, root's `package.json` and root's `turbo.json`.

It acts as the standard task for linting a package.

Then, we define the "_children_" tasks in `turbo.json`, pointing the correct `inputs`. We also add these "_children_" tasks to the `with` array of the `lint` task, so that they run in parallel.

```json
{
  "$schema": "https://turborepo.com/schema.json",
  "extends": ["//"],
  "tasks": {
    "lint": {
      "with": ["lint:ts", "lint:css"]
    },
    "lint:ts": {
      "inputs": ["**/*.ts", "**/*.tsx"]
    },
    "lint:css": {
      "inputs": ["**/*.css"]
    }
  }
}
```

### Result

All by running `pnpm run lint` from the root.

#### Scenario 1: No tasks are cached

<img width="1380" alt="image" src="https://github.com/user-attachments/assets/ce64211c-2eaa-4de8-b069-2920f5d40d90" />

#### Scenario 2: All tasks are cached

<img width="1380" alt="image" src="https://github.com/user-attachments/assets/61d935d0-bd43-491f-a829-78ef141b0006" />

#### Scenario 3: `lint:ts` passes (with cache), but `lint:css` fails

<img width="1380" alt="image" src="https://github.com/user-attachments/assets/c4fad381-c2ac-4332-9bcb-2c98b960b8a5" />
