# lib

## Overview
Shared helper libraries and configuration files used by Zed build, release, version-bumping, database migration linting, and deployment scripts across Bash, PowerShell, and Node.js.

## Interface
- `[blob-store.ps1:UploadToBlobStoreWithACL]`: PowerShell function accepting `-BucketName`, `-FileToUpload`, `-BlobStoreKey`, and `-ACL`.
- `[blob-store.ps1:UploadToBlobStorePublic]`: PowerShell function accepting `-BucketName`, `-FileToUpload`, and `-BlobStoreKey` (delegates to `UploadToBlobStoreWithACL` with `-ACL "public-read"`).
- `[blob-store.ps1:UploadToBlobStore]`: PowerShell function accepting `-BucketName`, `-FileToUpload`, and `-BlobStoreKey` (delegates to `UploadToBlobStoreWithACL` with `-ACL "private"`).
- `[blob-store.sh:upload_to_blob_store_with_acl]`: Bash function accepting positional arguments `bucket_name` (`$1`), `file_to_upload` (`$2`), `blob_store_key` (`$3`), and `acl` (`$4`).
- `[blob-store.sh:upload_to_blob_store_public]`: Bash function accepting `$1`, `$2`, `$3` and delegating to `upload_to_blob_store_with_acl` with `"x-amz-acl:public-read"`.
- `[blob-store.sh:upload_to_blob_store]`: Bash function accepting `$1`, `$2`, `$3` and delegating to `upload_to_blob_store_with_acl` with `"x-amz-acl:private"`.
- `[bump-version.sh:$1..$5]`: CLI positional arguments `package` (`$1`), `tag_prefix` (`$2`), `tag_suffix` (`$3`), `version_increment` (`$4`), and optional `gpui_release` (`$5`, defaults to `"false"`).
- `[deploy-helpers.sh:export_vars_for_environment]`: Bash function accepting `environment` (`$1`) and exporting variables from `crates/collab/k8s/environments/${environment}.sh`.
- `[deploy-helpers.sh:target_zed_kube_cluster]`: Bash function ensuring the active `kubectl` context is `do-nyc1-zed-1`.
- `[deploy-helpers.sh:tag_for_environment]`: Bash function mapping `environment` (`$1`: `production` or `staging`) to git/deploy tag (`collab-production` or `collab-staging`).
- `[deploy-helpers.sh:url_for_environment]`: Bash function mapping `environment` (`$1`: `production` or `staging`) to service URL (`https://collab.zed.dev` or `https://collab-staging.zed.dev`).
- `[release-notes.js:releaseNotesSection]`: Exported CommonJS function extracting the substring after the last `^Release Notes:` heading followed by a `- ` bullet.
- `[release-notes.js:releaseNotesEntries]`: Exported CommonJS function parsing bullet paragraphs starting with `- ` into normalized single-line string entries.
- `[squawk.toml:excluded_rules]`: Configuration array excluding PostgreSQL migration lint rules `prefer-identity`, `prefer-timestamptz`, `prefer-big-int`, and `prefer-bigint-over-int`.
- `[squawk.toml:pg_version]`: PostgreSQL target version string set to `"15.0"`.
- `[workspace.ps1:ParseZedWorkspace]`: PowerShell function populating `$env:ZED_WORKSPACE` and `$env:RELEASE_VERSION` from `cargo metadata`.

## Invariants
- `[blob-store.ps1:UploadToBlobStoreWithACL]`: Unconditionally overrides `$ACL` to `"public-read"` (`$ACL = "public-read"`), even when invoked from `UploadToBlobStore` with `-ACL "private"`.
- `[blob-store.ps1:UploadToBlobStoreWithACL]`: Requires environment variables `DIGITALOCEAN_SPACES_SECRET_KEY` and `DIGITALOCEAN_SPACES_ACCESS_KEY` for HMAC-SHA1 signature generation.
- `[blob-store.sh:upload_to_blob_store_with_acl]`: Requires environment variables `DIGITALOCEAN_SPACES_SECRET_KEY` and `DIGITALOCEAN_SPACES_ACCESS_KEY` for HMAC-SHA1 signature generation.
- `[bump-version.sh:set -eu]`: Runs with `errexit` and `nounset`; exits with status `1` if `git status --short --untracked-files=no` is non-empty.
- `[deploy-helpers.sh:export_vars_for_environment]`: Requires `crates/collab/k8s/environments/${environment}.sh` to exist as a regular file or exits with status `1`.
- `[deploy-helpers.sh:tag_for_environment]`: Only accepts `production` or `staging` as `$1`; otherwise exits with status `1`.
- `[deploy-helpers.sh:url_for_environment]`: Only accepts `production` or `staging` as `$1`; otherwise exits with status `1`.
- `[release-notes.js:releaseNotesSection]`: Matches only `Release Notes:` headings at the start of a line (case-insensitive, multiline) immediately followed by newlines and a `- ` bullet, selecting the last match in the text.
- `[release-notes.js:releaseNotesEntries]`: Stops parsing at the first blank-line-separated paragraph that does not begin with `- `, excluding trailing non-bullet paragraphs and commit trailers.

## Side Effects
- `[blob-store.ps1:UploadToBlobStoreWithACL]`: Reads `$FileToUpload` into a byte array and sends an HTTP PUT request via `Invoke-WebRequest` to `https://${BucketName}.nyc3.digitaloceanspaces.com/${BlobStoreKey}`.
- `[blob-store.sh:upload_to_blob_store_with_acl]`: Executes `curl` to send an HTTP PUT upload of `$file_to_upload` to `https://${bucket_name}.nyc3.digitaloceanspaces.com/${blob_store_key}`.
- `[bump-version.sh:cargo/brew]`: Installs `cargo-edit` via `cargo install cargo-edit` if `cargo-set-version` is missing, and installs `jq` via `brew install jq` if `jq` is missing.
- `[bump-version.sh:cargo/git]`: Updates crate version via `cargo set-version --package $package --bump $version_increment`, runs `cargo check --quiet`, commits tracked changes (`git commit --quiet --all`), and creates a local git tag (`git tag ${tag_name}`).
- `[deploy-helpers.sh:export_vars_for_environment]`: Exports all non-comment, non-blank variable assignments from `crates/collab/k8s/environments/${environment}.sh` into the current shell environment.
- `[deploy-helpers.sh:target_zed_kube_cluster]`: Runs `doctl kubernetes cluster kubeconfig save zed-1` to update local kubeconfig when the current `kubectl` context is not `do-nyc1-zed-1`.
- `[workspace.ps1:ParseZedWorkspace]`: Executes `cargo metadata --no-deps --offline` and mutates process environment variables `$env:ZED_WORKSPACE` and `$env:RELEASE_VERSION`.

## Verification
- Run: `./script/shellcheck-scripts`
