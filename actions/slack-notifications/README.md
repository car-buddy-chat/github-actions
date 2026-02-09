# CarBuddy Slack Notifications Action

A composite GitHub Action that sends Slack notifications for issues, pull requests, reviews, and comments across the CarBuddy organization.

## Features

✅ **Complete Coverage**: All GitHub events for issues and PRs
✅ **Beautiful Formatting**: Rich Slack block formatting
✅ **Proper Status Reporting**: Shows success/failure correctly in GitHub Actions
✅ **Easy to Use**: Single action for all notification types
✅ **Private Repo Friendly**: Works perfectly in private repositories

## Notifications Supported

### Pull Requests
- 🔀 PR Opened
- ✅ PR Merged
- ❌ PR Closed (not merged)
- 🔄 PR Reopened
- 👀 PR Ready for Review

### Reviews
- ✅ Review Approved
- 🔴 Changes Requested
- 💬 Review Comment

### Comments
- 💬 Review Comment on Code
- 💬 New PR Comment

### Issues
- 🆕 Issue Opened
- ✅ Issue Closed
- 🔄 Issue Reopened

## Usage

### For Pull Requests

```yaml
name: Slack PR Notifications

on:
  pull_request:
    types: [opened, closed, reopened, ready_for_review]
  pull_request_review:
    types: [submitted]
  pull_request_review_comment:
    types: [created]
  issue_comment:
    types: [created]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - uses: car-buddy-chat/slack-notifications-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### For Issues

```yaml
name: Slack Issue Notifications

on:
  issues:
    types: [opened, closed, reopened]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - uses: car-buddy-chat/slack-notifications-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `webhook-url` | Slack webhook URL for sending notifications | Yes |
| `notification-type` | Type of notification (auto-detected) | No |

## Setup

1. **Get Slack Webhook URL**:
   - Create an Incoming Webhook in your Slack workspace
   - Save the webhook URL

2. **Add Secret to Repository**:
   ```bash
   gh secret set SLACK_WEBHOOK_URL --body "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
   ```

3. **Create Workflow File**:
   - Copy one of the usage examples above
   - Save to `.github/workflows/slack-notify.yml`

4. **Test It**:
   - Create an issue or PR
   - Check Slack for notification
   - Verify workflow shows as "success" ✅

## Why Composite Action vs Reusable Workflow?

### Composite Actions ✅
- ✅ Proper status reporting (shows success/failure)
- ✅ Full step visibility in workflow runs
- ✅ Works in private repositories
- ✅ Easy debugging with complete logs
- ✅ No cross-repo permission issues

### Reusable Workflows ❌
- ❌ Status reporting broken for private repos
- ❌ Empty steps array in workflow runs
- ❌ No visible execution logs
- ❌ Shows "failure" even when working

## Migration from Reusable Workflows

If you're currently using the reusable workflows approach:

**Old (17 lines, but shows "failure")**:
```yaml
jobs:
  notify:
    uses: car-buddy-chat/carbuddy/.github/workflows/reusable-slack-prs.yml@main
    secrets: inherit
```

**New (4 lines, shows "success"**):
```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - uses: car-buddy-chat/slack-notifications-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Development

### Local Testing

You can test this action locally using [act](https://github.com/nektos/act):

```bash
act pull_request -e test-event.json
```

### Contributing

1. Fork the repository
2. Make your changes
3. Test thoroughly
4. Create a pull request

### Releasing

To create a new version:

```bash
git tag v1.0.0
git push origin v1.0.0
```

Users can reference by tag:
- `@v1` - Latest v1.x.x (recommended)
- `@v1.0.0` - Specific version
- `@main` - Latest (not recommended for production)

## Troubleshooting

### Notifications not appearing

1. **Check webhook URL**: Verify the URL is correct and not expired
2. **Check secrets**: Ensure `SLACK_WEBHOOK_URL` is set correctly
3. **Check workflow logs**: Look for error messages in the step output
4. **Test webhook**: Try sending a test curl request to the webhook

### Workflow shows "skipped"

This means none of the conditions matched. Check:
- Event type matches workflow triggers
- Conditional logic (PR vs Issue detection)

### Multiple notifications for one event

Each step has specific conditions. If you see duplicates:
- Check that event triggers are correct
- Verify conditional logic isn't overlapping

## Support

For issues or questions:
- Create an issue in this repository
- Check existing issues for similar problems
- Review the investigation docs in the evhc repo

## License

Private - CarBuddy Organization Only

## Related Documentation

- [Workflow Investigation Findings](../evhc/WORKFLOW_INVESTIGATION_FINDINGS.md)
- [Alternative Solutions Research](../evhc/ALTERNATIVE_SOLUTIONS_RESEARCH.md)
- [GitHub Composite Actions Docs](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
