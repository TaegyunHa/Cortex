## Atlassian CLI

### Install
> https://developer.atlassian.com/cloud/acli/guides/install-acli/

## CLI

### Create Work Items

Creates new Jira work items to track work. Supports creating single or multiple work items with various field configurations.

#### Basic Usage
```bash
# Create a basic work item
acli jira workitem create --project PROJECT_KEY --type Task --summary "Task description"

# Create with assignee and description
acli jira workitem create --project DEV --type Bug --summary "Fix login issue" \
  --assignee "@me" --description "Users cannot login with SSO"

# Create subtask under parent
acli jira workitem create --project DEV --type Subtask --summary "Update tests" \
  --parent DEV-123
```

#### Key Options
- `-p, --project` - Project key (required)
- `-t, --type` - Work item type (e.g., Task, Bug, Story)
- `-s, --summary` - Work item summary (required)
- `-a, --assignee` - Assignee email or '@me' for self-assign
- `-d, --description` - Description in plain text or ADF
- `--parent` - Parent work item key for subtasks
- `--label` - Add labels to the work item
- `--from-json` - Create from JSON file for bulk creation
- `--json` - Output created work item as JSON

### Edit Work Items

Modifies existing Jira work items. Can edit single or multiple work items using various selection methods.

#### Basic Usage
```bash
# Edit a single work item
acli jira workitem edit --key DEV-123 --summary "Updated summary"

# Edit multiple work items using JQL
acli jira workitem edit --jql "project = DEV AND status = 'To Do'" \
  --assignee "@me" --labels "urgent,review"

# Bulk edit with confirmation
acli jira workitem edit --filter 12345 --type Story --yes

# Remove assignee from work item
acli jira workitem edit --key DEV-456 --remove-assignee

# Edit from JSON file for complex updates
acli jira workitem edit --from-json updates.json
```

#### Options

```
-a, --assignee string           Assign work item with email or account ID. Use '@me' to self-assign, 'default' to assign to the project's default assignee
-d, --description string        Edit the description in plain text or Atlassian Document Format (ADF)
  --description-file string   Read the description in plain text or Atlassian Document Format (ADF) from the file
  --filter string             Filter ID of work items to be edited
  --from-json string          Read the work item definition from a JSON file
  --generate-json             Generates a JSON file that could be used for work item editing
-h, --help                      Show help for command
  --ignore-errors             Ignore the errors and continue
  --jql string                JQL query for work items to be edited
  --json                      Generate a JSON output
-k, --key string                A list of work item keys to be edited
-l, --labels string             Edit the labels
  --remove-assignee           Remove the assignee
  --remove-labels string      Remove the labels
-s, --summary string            Edit the summary
-t, --type string               Edit the work item type
-y, --yes                       Confirm edit without prompting
```

### Link Work Items

Manages relationships between Jira work items. Supports creating, deleting, and listing links.

#### Subcommands
- `create` - Create a new link between work items
- `delete` - Remove an existing link
- `list` - List links for a work item
- `type` - List available link types

#### Basic Usage
```bash
# Create parent-child relationship
acli jira workitem link create --type Parent --in "DEV-123" --out "DEV-456"

# Create blocks relationship
acli jira workitem link create --type Blocks --in "DEV-789" --out "DEV-101"

# List all links for a work item
acli jira workitem link list --key DEV-123

# Delete a specific link
acli jira workitem link delete --id LINK_ID

# Show available link types
acli jira workitem link type
```

#### Common Link Types
- `Parent` / `Child` - Hierarchical relationships
- `Blocks` / `Is Blocked By` - Dependency relationships
- `Relates To` - General relationship
- `Duplicates` / `Is Duplicated By` - Duplicate tracking
- `Clones` / `Is Cloned By` - Clone relationships

### Transition Work Items

Moves work items through workflow statuses. Supports single or bulk transitions.

#### Basic Usage
```bash
# Transition single work item to specific status
acli jira workitem transition --key DEV-123 --status "In Progress"

# Bulk transition using JQL
acli jira workitem transition --jql "project = DEV AND assignee = currentUser()" \
  --status "Done" --yes

# Transition with filter
acli jira workitem transition --filter 12345 --status "Ready for Review"

# Transition with error handling for bulk operations
acli jira workitem transition --jql "fixVersion = '2.0'" \
  --status "Testing" --ignore-errors
```

#### Key Options
- `-k, --key` - Work item key(s) to transition
- `--jql` - JQL query to select work items
- `--filter` - Filter ID to select work items
- `-s, --status` - Target status name
- `-y, --yes` - Skip confirmation prompt
- `--ignore-errors` - Continue on errors in bulk operations
- `--json` - Output results as JSON

### View Work Items

Retrieves and displays work item information. Supports viewing single or multiple work items with field selection.

#### Basic Usage
```bash
# View single work item details
acli jira workitem view --key DEV-123

# View specific fields only
acli jira workitem view --key DEV-456 --fields "summary,status,assignee"

# View as JSON for parsing
acli jira workitem view --key DEV-789 --json

# Open work item in browser
acli jira workitem view --key DEV-101 --web

# View multiple work items
acli jira workitem view --key "DEV-123,DEV-456,DEV-789"
```

#### Key Options
- `-k, --key` - Work item key(s) to view (comma-separated for multiple)
- `--fields` - Comma-separated list of fields to display
- `--json` - Output as JSON for machine processing
- `--web` - Open work item in web browser
- `--verbose` - Show all available fields