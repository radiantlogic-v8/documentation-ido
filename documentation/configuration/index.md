# Startup Guide

## Data sources requirements

- Identify the entities you will be working with. Start simple at first by focusing on accounts and groups.
- Select the data sources you will be using:
  - LDAP or AD are the most straight forward to get started with.
  - Custom data sources are supported through the use of virtualization and periodic cache refresh in Identity Data Management as an intermediary layer. For the ones that support a solid change detection, we are working on a native, no-copy approach similar to what is done with LDAP and AD.
- For entities mapped in Identity Observability, a **invariant ID** is required. A DN is not a good choice as it can change through modifyRdn operations. You must use internal identifiers like objectGUID for AD, UUID or EntryUUID for LDAP or last resort, a custom combination of fields to compose a stable identifier.

## Understand the model

As you're getting ready, identify the entities you will be working with. Start simple at first by focusing on core objects like **accounts** and **groups**.

The JSON representation of the model is self explanatory once you know the terminology:

- **Concepts** equivalent to vertices, is a logical grouping of tags, a main tag and optionally additional tags. The schema lists the incoming and outgoing edges.
- **Tags** logical grouping of properties.
- **Edges** Relationship between concepts/vertices. They can host their own properties.
- **Properties** key-value pair associated to concepts and edges.
- **external_identifier** this is the identifier that should allow to join the entity with other sources of data. In LDAP access logs, it is usually the dn.
- **displayname** this is mandatory since it's used as the entity name in the Identity Observability UI.
- **dn** this is automatically fed by the pipeline engine.

## Prepare the configuration templates

The configuration templates are a way to configure the orchestrator with reusable pieces of configuration. See the [graph pipeline template configuration](./template-configuration.md) for more details.

## Deploy the configuration and monitor

Identity Data Management and Identity Observability's orchestrator are the main components you will be working with at first in this order:

1. Identity Data Management:
    - Create the data sources.
    - Optionally, modify the LDAP schema to support the object classes and their attributes if they are not part of Identity Data Management out of the box.
    - If the datasource doesn't support change detection, create a virtual view and setup a periodic cache refresh for it. This cached naming context can then be used as an LDAP datasource to feed the Identity Observability pipelines.
2. Identity Observability's orchestrator:
    - Create the configuration templates. Ideally, one per data source.
    - Apply the templated configuration (the master configuration referencing and extending the templates).
    - Monitor the orchestrator metrics to ensure the data is flowing through the pipeline.
    - Monitor the graph pipelines with Grafana to ensure they are running as expected.

## Understand the flow

When deploying a new configuration, the orchestrator will automatically:

- Validate the configuration is valid and aligned with the graph schema.
- Verify the referenced datasources exist.
- Validate the attributes and object names are part of the datasource schema.
- Create and add flat views in the identity catalog naming context for each object configured.
- Create a default connector configuration for Identity Observability.
- If the connector configuration is complete, an initial upload is triggered automatically. Connectors configuration are complete when they don't need human intervention to customize them.
- If the pipeline is not uploaded automatically, the pipeline state is flagged as `CONFIGURATION_INCOMPLETE`.
- After the upload of a pipeline is finished, the connector for that pipeline is restarted automatically.
- Create and deploy the graph pipelines which are pushing the normalized data into the graph database following the configuration.
- The orchestrator keeps on monitoring the state of the pipelines and the connectors which is reported in the prometheus metrics endpoint.

When applying a modified configuration, the orchestrator will:

- Validate the configuration changes are supported: only non structural changes are supported like the addition of a new source objects, the addition of new edges, added or removed properties. Changing the datasource or the object name of an existing source object is considered a structural change and is not supported.
- Validate the configuration is aligned with the graph schema and the datasources schema.
- Deploy/Undeploy the views in the identity catalog needed to supported the new configuration.
- Identify the dependencies that need to be re-uploaded which is common when adding a vertex or an edge that are based on a source object that was already processed in a previous run.
- Re-upload the dependencies and the pipeline.
- Backup the graph pipelines are redeploy them based on the new configuration.

When applying an empty configuration, the orchestrator will:

- Undeploy the identity catalog views.
- Undeploy the connectors.
- Undeploy the graph pipelines.
- Empty the graph database from any data.
