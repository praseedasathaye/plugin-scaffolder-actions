# CNOE Scaffolder Actions Plugin

The "cnoe-io/plugin-scaffolder-actions" repository is a collection of extended or custom actions for the Backstage Scaffolder plugin, designed to enhance the developer experience and enable more customized scaffolding workflows within the Backstage ecosystem. The actions are designed to be used alongside the built-in actions provided by the Backstage Scaffolder plugin. The actions are added to the list of available actions when setting up the Scaffolder in a Backstage application. By providing these custom actions, the repository allows Backstage users to extend the Scaffolder functionality to support their specific use cases and requirements, such as resource sanitization, dependency verification, and Kubernetes manifest deployment.

Here are the three custom actions that can be used in Backstage Scaffolder templates:

- ```cnoe:utils:sanitize```: Sanitizes (removes empty fields) from resources before further processing.
- ```cnoe:verify:dependency```: Verifies dependencies for CNOE resources.
- ```cnoe:kubernetes:apply```: Applies Kubernetes manifests to a template.

## Getting Started

Add to your Backstage app.
```bash
# From your Backstage root directory
yarn add --cwd packages/backend @cnoe-io/plugin-scaffolder-actions
```
```bash
# To be able to keep using the built-in actions.
yarn add --cwd packages/backend @backstage/integration
```

Append it to your existing actions in `packages/backend/src/plugins/scaffolder.ts`
```typescript
import { CatalogClient } from '@backstage/catalog-client';
import { createRouter, createBuiltinActions } from '@backstage/plugin-scaffolder-backend';
import { ScmIntegrations } from '@backstage/integration';
import { Router } from 'express';
import type { PluginEnvironment } from '../types';
import {
  createSanitizeResource,
  createVerifyDependency,
  createKubernetesApply,
} from "@cnoe-io/plugin-scaffolder-actions";

export default async function createPlugin(
  env: PluginEnvironment,
): Promise<Router> {
  const catalogClient = new CatalogClient({ discoveryApi: env.discovery });
  const integrations = ScmIntegrations.fromConfig(env.config);

  const builtInActions = createBuiltinActions({
    integrations,
    catalogClient,
    config: env.config,
    reader: env.reader,
  });
  
  const cnoeActions = [
    createSanitizeResource(),
    createVerifyDependency(),
    createKubernetesApply(env.config),
  ]

  const actions = [
      ...builtInActions,
      ...cnoeActions,
  ]

  return await createRouter({
    actions,
    catalogClient,
    logger: env.logger,
    config: env.config,
    database: env.database,
    reader: env.reader,
    identity: env.identity,
  });
}
```

Done! You can now use any of the action in your software templates.
```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: hello-world-on-kubernetes
  title: Hello World on Kubernetes
spec:
  steps:
    - id: sanitize-resource
      name: Sanitize Resource
      action: cnoe:utils:sanitize
      input:
        resource: ${{ serialize.output }}
```

## List of Actions

Here is a list of running actions.

| Action                 | id                     | Description                                                        | Reference                            |
|------------------------|------------------------|--------------------------------------------------------------------|--------------------------------------|
| createKubernetesApply  | `cnoe:kubernetes:apply`  | Apply Kubernetes manifest to a template                            | [k8s-apply](src/actions/k8s-apply.ts) |
| createVerifyDependency | `cnoe:verify:dependency` | Verify resource dependencies for CNOE                              | [verify](src/actions/verify.ts)      |
| createSanitizeResource | `cnoe:utils:sanitize`    | Sanitize resources (remove empty fields) before further processing | [sanitize](src/actions/sanitize.ts)  |

For more detailed information about these actions, go to `/create/actions` endpoint of your Backstage instance after installing these actions. 
If you are running this locally, the endpoint should be `http://localhost:3000/create/actions`
