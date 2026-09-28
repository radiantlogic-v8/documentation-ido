# Tags

Tags are used to label and organize identity objects across the platform, including agents. Use tags to group objects by environment, owner, department, cost center, risk level, or another classification that matters to your organization.

Tags help you:

- Find and filter objects in dashboards, lists, and Query Builder.
- Scope observations and controls to groups of objects.
- Report on ownership, deployment, status, and other classifications.

Tag names are unique platform-wide, so each tag has a consistent meaning wherever it appears.

## Tag types

| Type | Assigned by | Editable | Available for policy scoping |
|---|---|---|---|
| Manual tag | Administrator | Yes | Yes |
| Dynamic tag | Observation or supported control rule | No | Yes |
| Backend label | Source platform through a connector | No | No |

Use a **manual tag** for classifications that require human judgment, such as an approved exception, business owner, or criticality level.

Use a **dynamic tag** for classifications derived from object attributes that can change over time, such as `env:production`, `data:regulated`, or `ownership:unassigned`.

Use **backend labels** to view, search, and correlate metadata imported from the source platform.

## Manual tags

Administrators can add or remove manual tags from individual identity objects or groups of objects.

### Tag one object

1. Open the [object details](../object-details/overview/)) page in the Identity Observability portal.
2. In **Tags**, select the tag editor.
3. Enter a tag name, then select an existing suggestion or create a new tag.
4. To remove a tag, select the **x** next to it.

### Tag multiple objects

1. In [Query Builder](../query-builder/overview/), run a query for the objects you want to update.
2. Select one or more results.
3. Select **Bulk tag**.
4. Add or remove manual tags.

Bulk tagging applies only to manual tags. Dynamic tags are assigned by rules and are not available in the bulk tag selector.

## Dynamic tags

Dynamic tags are assigned automatically by observation rules or supported controls. The platform adds a dynamic tag when an identity object matches the rule and removes it when the object no longer matches.

### Create a dynamic tag

1. Create or edit an [observation](../observations/creating-observations/), or a [control](../controls/creating-controls/).
2. Define criteria that identify the objects to tag.
3. In the tag configuration, specify one or more tags.
4. Save the observation or control.

Dynamic tags represent an object's current state, not its historical state.

### Identify the rule

The tag administration page lists the observation rules associated with each dynamic tag, including the rule name and identifier. Use this information to determine why an object has a tag or to update the rule that assigns it.

## Backend labels

Backend labels are metadata imported from the source platform through a connector, such as cloud-resource tags.

You can use backend labels to:

- View source metadata in an object's **Provider details**.
- Search and filter objects by label key or value.
- Correlate objects with their source-platform configuration.

Backend labels are read-only. You cannot edit them directly to scope an observation or control.

To group objects based on a backend label, create a dynamic-tag rule that matches the label or apply a manual tag to the matching objects.

## Where to use tags

| Location | How tags are used |
|---|---|
| Object details | View and manage supported tags for an identity object, including agents. |
| Dashboards | Filter dashboard content by platform and tag. Some dashboard classifications, such as agent type, can be tag-based. |
| Query Builder | Find objects by tag, create reports, and select objects for bulk actions. |
| Observations and controls | Scope a policy to a tagged group rather than maintaining a list of individual objects. |

## FAQ

**Can I remove a dynamic tag from one object?**

No. A dynamic tag is removed only when the object no longer matches its assigning rule. Update the observation or control criteria to change the result.

**Why did a dynamic tag disappear?**

The object no longer matches the rule that assigned the tag. Review the associated rule on the tag administration page.

**Can an object have manual and dynamic tags?**

Yes. An identity object can have both. Dynamic tags display a lock indicator to distinguish them from editable manual tags.

**Can I scope a policy with a backend label?**

No. Create a dynamic tag that matches the backend label or apply a manual tag to the applicable objects.
