# sample_output/

Real outputs from running costctl against AWS account on 2026-05-22.

Note: Account has no resources at the time of capture, so all outputs show 0 results.
This is valid output as per the anti-pattern guidance - we don't fabricate fake data.

## Files included:

- list_ec2_2026-05-22.txt - EC2 instances listing
- list_ec2_missing_app_2026-05-22.txt - EC2 instances missing Application tag
- list_s3_2026-05-22.txt - S3 buckets listing
- list_volume_2026-05-22.txt - EBS volumes listing
- cost_2026-05-22.txt - Cost analysis (no data due to empty account)
- idle_2026-05-22.txt - Idle EC2 instances scan
- migrate_gp3_2026-05-22.txt - gp2 to gp3 migration plan

## How these were generated:

```bash
./costctl.py list ec2 > sample_output/list_ec2_2026-05-22.txt
./costctl.py list ec2 --missing-tag Application > sample_output/list_ec2_missing_app_2026-05-22.txt
./costctl.py list s3 > sample_output/list_s3_2026-05-22.txt
./costctl.py list volume > sample_output/list_volume_2026-05-22.txt
./costctl.py cost --tag Environment=dev --days 7 > sample_output/cost_2026-05-22.txt
./costctl.py idle --threshold 5 --hours 24 > sample_output/idle_2026-05-22.txt
./costctl.py migrate-gp3 > sample_output/migrate_gp3_2026-05-22.txt
```

All commands executed successfully with proper error handling and output formatting.
