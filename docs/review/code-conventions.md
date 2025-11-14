## Test Builders

We’ve agreed to use test builders as long as we keep them organised and clean. They allow us to create complex objects for our tests in a clean way, helping to avoid code duplication.

Our recommended approach is to have one builder per domain model. For a practical example, please check the `UserData` class in the `solo-user-service` project ([link](https://github.com/vwdigitalhub/solo-user-service/blob/develop/src/test/kotlin/com/vwg/solo/user/service/UserData.kt)).

## Naming conventions

To prevent naming conflicts with domain model classes, we have established the following naming conventions for documents and response objects:

- **For documents:** Add the `Document` suffix (e.g. `UserDocument`)
- **For document-related classes:** Add the `DB` suffix (e.g. `RoleDB`)
- **For responses:** Add the `Response` suffix (e.g. `UserResponse`)
- **For response-related classes:** Add the `RS` suffix (e.g. `RoleRS`)

## SBOM Inventory

**Naming:**

To comply with the documentation and align with the component naming convention, projects should be built as follows:

- We will have one component per version and microservice: {microservice} + ":" + {version}
- Each environment must add a prefix (except prod):
    - dev: -build -> `solo-rating-service:0.4.0-SNAPSHOT-build`
    - qa: -rc -> `solo-rating-service:0.4.0-rc`
    - prod: no prefix -> `solo-rating-service:0.4.0`

**Disable previous version:**

To have a more readable and maintainable dashboard with less active versions, if a new component version is created, the previous version must be disabled.

Practical case: If we have a version solo-rating-service:0.4.0-SNAPSHOT-build in the development environment and we have created the following version solo-rating-service:0.5.0-SNAPSHOT-build, the version solo-rating-service:0.4.0-SNAPSHOT-build will have to be disabled as is described in the following process:

1. Access to [solo](https://ui.sbom-inventory.vwapps.run/ns/423dfdb0-9465-4592-8a7f-50559578cc13) project in the [SBOM Inventory dashboard](https://ui.sbom-inventory.vwapps.run/) through the [service board](https://service-board.vwapps.run/projects/de25b650-71a5-4b5f-beee-da4ce82890f0/overview/services).
2. In `Active Deployments`, select the old version (if there are multiple versions, you can filter by component):
3. In the `Stage` column, the currently deployed environment will appear with a drop-down menu. Click on it and select the `Deactivate` option.