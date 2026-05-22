# Reflections

## 1. Multi-account

To run costctl against 100 AWS accounts, I would implement:

Cross-account IAM roles with a central orchestrator account. Each target account has a CostCtlRole that trusts the central account. The tool uses sts.assume_role() to switch contexts.

Configuration via accounts.yaml listing account IDs, names, and role ARNs. Add CLI flags like --account prod or --all-accounts.

Parallel execution using ThreadPoolExecutor to query 10-20 accounts simultaneously, reducing total runtime from 8+ minutes to under 1 minute.

Aggregated CSV output with columns: account_id, account_name, resource_type, resource_id, state, tags, monthly_cost. This enables cross-account analysis in Excel or BI tools.

Alternative approach: loop through AWS CLI profiles, but this is less secure and harder to maintain than cross-account roles.

## 2. idle vs Trusted Advisor

Trust idle (24h window) when:
- Need quick response for short-term workloads like batch jobs or ML training that finished but weren't terminated
- Dev/test environments with predictable patterns (8h workdays, idle weekends)
- Custom thresholds needed (idle supports --threshold flag, TA doesn't)
- Immediate action required after sprint completion or budget alerts

Trust Trusted Advisor (14 days) when:
- Production workloads with fluctuating traffic patterns
- Avoiding false positives from newly launched instances or scheduled tasks
- Compliance and audit requirements (TA is AWS official recommendation)
- CloudWatch metrics have gaps or are unavailable

Best practice: combine both. Use idle for weekly quick scans, TA for monthly deep analysis. Only terminate instances flagged by both tools.

## 3. clean --apply blast radius

To limit damage from accidental clean in a shared account:

Preventive controls:
- Dry-run by default (already implemented)
- Double confirmation prompt when >10 resources targeted
- Protected tags: refuse to delete resources tagged keep=true or protected=true
- Account whitelist: only allow clean in dev/sandbox accounts, block in production
- IAM permissions boundary denying termination of production-tagged resources

Detective controls:
- Audit logging to CloudWatch Logs with user, timestamp, account, and targets
- SNS notifications to team Slack channel on any clean --apply execution

Corrective controls:
- Auto-backup: create AMI snapshots before terminating instances
- Undo command to restore from recent backups
- Time restrictions: only allow clean during business hours when team is online

Most important: dry-run default, protected tags, and auto-backup before delete.

## 4. AI assistance

Approximately 85% of code came from AI (Claude) unmodified:
- All boilerplate (imports, function signatures, boto3 client initialization)
- AWS API calls and pagination logic
- Helper function usage (parse_kv, tags_to_dict, tags_match)
- Error handling structure

I actively modified 15%:
- Output formatting: AI didn't match README's exact column alignment and spacing requirements
- Edge cases: added state filtering for terminated instances, handled empty S3 buckets correctly
- Safety checks: AI didn't add the "refuse non-empty bucket" logic for S3 deletion
- Type consistency: ensured tags are always dict not list for consistency with helpers

AI was effective because:
- Detailed docstrings provided clear specifications
- Test cases defined exact expected behavior
- Standard AWS APIs that AI has seen extensively

I still needed to review and fix edge cases. AI saved ~70% of implementation time but cannot replace understanding the requirements and testing thoroughly.

## 5. W7 carry-over

Keep for production:
- list: essential for inventory, compliance audits, security scans. Add multi-account, multi-region, CSV output.
- cost: critical for visibility. Add trend analysis, budget alerts, BI tool integration.
- tag: foundation of cost management. Add bulk tagging, tag validation, enforcement.
- idle: valuable for waste detection. Integrate with Trusted Advisor, add rightsizing recommendations.

Modify heavily:
- terminate: too dangerous as-is. Add approval workflows, cooling-off periods, MFA confirmation, disable in production accounts.
- clean: extreme blast radius. Restrict to dev accounts only, require multi-person approval, auto-backup before delete.

Drop:
- migrate-gp3: too risky for production automation. Volume modifications can impact performance. Better handled manually with change management or using AWS Compute Optimizer recommendations.

Key principle: W7 production needs observability and governance, not automation of destructive actions. Read-only commands (list, cost) are safe and valuable. Write commands (terminate, clean) need heavy restrictions or should be dropped entirely.
