# Terraform-Modules-For-Compliance

# compliant-gcs-bucket

Terraform module that provisions a NIST 800-53 compliant GCS bucket on GCP.
All compliance controls are hardcoded inside the module — consumers cannot
disable them.

## Controls enforced

| Control | Enforcement |
|---------|-------------|
| SC-12   | Customer-managed KMS keyring and crypto key (we own the key, not Google) |
| SC-13   | 90-day automatic key rotation via rotation_period = "7776000s" |
| SC-28   | CMEK encryption set as default_kms_key_name on the bucket |
| AU-11   | Object versioning enabled + retention_policy enforced per environment |
| CM-6    | Four required labels (project, environment, managed_by, compliance_scope) |
| AC-3    | uniform_bucket_level_access = true, public_access_prevention = "enforced" |

## Usage

```hcl
module "data_bucket" {
  source = "../../modules/compliant-gcs-bucket"

  gcp_project        = "your-gcp-project"
  project_label      = "cgep-lab"
  environment        = "dev"
  retention_days     = 30
  bucket_name_suffix = "dev-data-001"
}

output "attestation" { value = module.data_bucket.compliance_attestation }
```

## Important: two location variables

- `location` — GCS bucket location. Accepts multi-regions (US, EU).
- `kms_location` — KMS keyring location. Must be a single region (us-central1, etc.).
  Setting both to the same multi-region fails with KMS_RESOURCE_NOT_FOUND_IN_LOCATION.

## Outputs

- `bucket_url` — gs:// URL
- `bucket_self_link` — self-link URI
- `kms_key_id` — CMEK key resource ID
- `compliance_attestation` — map used as downstream evidence in Lab 3 (Rego) and Lab 6 (OSCAL)
