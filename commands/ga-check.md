name: ga-check
description: Investigate failed GitHub Actions workflows and identify root causes
instructions: |
This command checks GitHub Actions workflow execution status and investigates the root cause of any failures.

## Execution Steps

### 1. Check Latest Workflow Run for Current Branch

```bash
# Get current branch name
CURRENT_BRANCH=$(git branch --show-current)

# Check the latest run for current branch
gh run list --branch "$CURRENT_BRANCH" --limit 1 --json conclusion,status,name,databaseId,createdAt,headBranch
```

- Check the most recent run for the current branch only
- Focus on "failure" or "cancelled" status
- Record run ID and workflow name
- If no runs found for current branch, expand search:
  ```bash
  # Fallback: check recent runs across all branches
  gh run list --limit 5 --json conclusion,status,name,databaseId,createdAt,headBranch
  ```

### 2. Inspect Failed Workflow Details

For any failed runs found:

```bash
gh run view <RUN_ID> --json jobs,conclusion,status
```

- Identify which jobs failed
- Note the failed step names

### 3. Extract and Analyze Error Logs

```bash
# First check logs for failed jobs only
gh run view <RUN_ID> --log-failed
```

Apply appropriate grep patterns based on error type:

- Test failures: `gh run view <RUN_ID> --log | grep -E "(FAIL|ERROR|Failed|Error:|✗|FAILED|AssertionError|TestFailure)"`
- Build errors: `gh run view <RUN_ID> --log | grep -E "(error:|Error:|ERROR|Build failed|compilation|cannot find|undefined reference)"`
- Dependencies: `gh run view <RUN_ID> --log | grep -E "(npm ERR|yarn error|bundle install|pip install.*failed|gem install|cargo build.*error)"`
- Timeouts: `gh run view <RUN_ID> --log | grep -E "(timeout|Timeout|TIMEOUT|exceeded|deadline)"`
- Network issues: `gh run view <RUN_ID> --log | grep -E "(ECONNREFUSED|ETIMEDOUT|SSL|certificate|connection refused|network)"`

### 4. Root Cause Investigation

#### 4.1 Analyze Error Context

- Extract full error messages with context (5 lines before/after):
  ```bash
  gh run view <RUN_ID> --log | grep -B 5 -A 5 -E "error|failed|Error"
  ```
- Identify stack traces and relevant file paths
- Note specific test names or build targets that failed

#### 4.2 Check Common Failure Patterns

- **Local Reproducibility**: Can this be reproduced locally? If not, it strongly indicates CI configuration issues, environment-specific problems, or missing secrets/variables
- **Missing Environment Variables**: Verify secrets and env vars are properly configured
- **Dependency Issues**: Check package.json, Gemfile, requirements.txt, go.mod, Cargo.toml
- **API Rate Limits**: GitHub API, npm registry, or other external service limits
- **Concurrency Issues**: Race conditions from parallel jobs
- **Flaky Tests**: Intermittent failures that might need retry logic
- **Resource Constraints**: Memory, disk space, or CPU limits
- **Version Mismatches**: Node, Python, Ruby, or other runtime versions

#### 4.3 Examine Related Files

Check as needed:

- Workflow file: `.github/workflows/<workflow-name>.yml`
- Recent commits: `git log --oneline -10`
- Changed files: `git diff HEAD~1 --name-only`
- Test files mentioned in errors
- Configuration files (jest.config.js, pytest.ini, etc.)

### 5. Generate Investigation Report

Format the findings as:

```markdown
## GitHub Actions Investigation Report

### Failed Workflow Summary

- **Workflow**: [name]
- **Run ID**: [id]
- **Failed at**: [timestamp]
- **Failed job(s)**: [job names]
- **Trigger**: [push/pull_request/schedule/manual]
- **Branch**: [branch name]

### Error Details
```

[Primary error messages and stack traces]

````

### Root Cause Analysis
[Detailed analysis of the failure]

### Recommended Solutions
1. **Immediate Fix**: [Quick resolution]
2. **Long-term Solution**: [Permanent fix]
3. **Prevention**: [How to avoid in future]

### Commands to Execute
```bash
# Suggested fix commands
````

### References

- [Workflow run URL]
- [Related PRs/Issues]
- [Documentation links]

## Additional Considerations

### Authentication & Access

- Verify GitHub CLI auth: `gh auth status`
- For private repos, ensure proper permissions
- Use `--repo owner/repo` flag if not in repo directory

### Performance Optimization

- For large logs: `gh run view <RUN_ID> --log > workflow.log` then analyze file
- Use `tail -n 1000` for recent log lines only
- Consider `--log-failed` to reduce log volume

### Recovery Actions

- Suggest re-run if transient: `gh run rerun <RUN_ID>`
- For failed jobs only: `gh run rerun <RUN_ID> --failed`
- Check if workflow has retry logic configured

## Error Handling

- If `gh` is not available: Guide GitHub CLI installation
- For permission errors: Explain required scopes (repo, workflow)
- For network issues: Suggest connectivity checks
- For parsing errors: Fall back to web UI investigation

## Output Expectations

Always provide:

1. Clear problem statement
2. Evidence from logs
3. Root cause hypothesis
4. Actionable solutions
5. Prevention strategies

```

```
