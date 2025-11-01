Title: checksum mismatch for github.com/charmbracelet/glamour v0.10.0

Body:
```
Summary
-------
When attempting to download/build `github.com/charmbracelet/glamour@v0.10.0` Go reports a checksum mismatch between the downloaded archive and the checksum recorded by the checksum server (sum.golang.org).

This causes downstream builds (for example `go install github.com/cli/cli/v2/cmd/gh@latest`) to fail with a SECURITY ERROR.

Evidence
--------
- sum.golang.org records for v0.10.0:
  - `h1:MtZvfwsYCx8jEPFJm3rIBFIMZUfUJ765oX8V6kXldcY=`

- Example download checksum reported by Go (from a failing run):
  - `downloaded: h1:41/IYxsmIpaBjkMXjrjLwsHDBlucd5at6tY5n2r/qn4=`

Reproduction steps
------------------
1. Check Go environment:
   ```bash
   go env GOPROXY GOSUMDB GOPRIVATE GONOSUMDB
   ```
2. Clear local cache and attempt direct download:
   ```bash
   go clean -modcache
   GOPROXY=direct go mod download -json github.com/charmbracelet/glamour@v0.10.0
   ```

Expected result
---------------
- `go mod download -json` should report `Sum: "h1:MtZvf..."` and the download should verify correctly.

Actual result
-------------
- The Go tool sometimes reports a downloaded checksum different from the sum.golang.org value (see "downloaded:" above), leading to a SECURITY ERROR.

Additional info I recorded
--------------------------
- The `Origin` info from `go mod download -json` (when successful) shows:
  - `Ref`: `refs/tags/v0.10.0`
  - `Hash`: `05ee9b5f4dcf3e4426c4ba41e1f9d7ea4f34d603`

What I would like you to confirm
--------------------------------
1. Are the contents of tag `v0.10.0` in this repository the canonical contents that sum.golang.org hashed? If so, can you confirm the commit/tag hash matches the one above?
2. If the tag was re-written or republished, please restore the original contents or publish a new patch release so the checksum server can record the correct value.

Attachments
-----------
- Paste the output of `GOPROXY=direct go mod download -json github.com/charmbracelet/glamour@v0.10.0` if it reports a different Sum to help triage.

Thank you — please let me know what extra information would help (I can attach `go env` output and the full `go mod download -json` output).
```

Diagnostics I collected while reproducing this locally
-----------------------------------------------------

These are the outputs I recorded in a devcontainer while attempting the direct download. They show the environment, the `go mod download -json` result and the SHA256 I computed for the downloaded zip file.

1) `go env GOPROXY GOSUMDB GOPRIVATE GONOSUMDB`

```
https://proxy.golang.org,direct
sum.golang.org
```

2) `GOPROXY=direct go mod download -json github.com/charmbracelet/glamour@v0.10.0`

```
{
  "Path": "github.com/charmbracelet/glamour",
  "Version": "v0.10.0",
  "Info": "/go/pkg/mod/cache/download/github.com/charmbracelet/glamour/@v/v0.10.0.info",
  "GoMod": "/go/pkg/mod/cache/download/github.com/charmbracelet/glamour/@v/v0.10.0.mod",
  "Zip": "/go/pkg/mod/cache/download/github.com/charmbracelet/glamour/@v/v0.10.0.zip",
  "Dir": "/go/pkg/mod/github.com/charmbracelet/glamour@v0.10.0",
  "Sum": "h1:MtZvfwsYCx8jEPFJm3rIBFIMZUfUJ765oX8V6kXldcY=",
  "GoModSum": "h1:f+uf+I/ChNmqo087elLnVdCiVgjSKWuXa/l6NU2ndYk=",
  "Origin": {
    "VCS": "git",
    "URL": "https://github.com/charmbracelet/glamour",
    "Hash": "05ee9b5f4dcf3e4426c4ba41e1f9d7ea4f34d603",
    "Ref": "refs/tags/v0.10.0"
  }
}
```

3) SHA256 of the downloaded zip (raw):

```
base64: mAYG4Jmj9JvL8lIfBU5SBf3NCNtJmXXNw3TF0PFMebg=
hex: 980606e099a3f49bcbf2521f054e5205fdcd08db499975cdc374c5d0f14c79b8
```

Notes about the above
---------------------
- The `Sum` field shown by `go mod download -json` is the canonical module checksum (and matches the value in `go.sum` and on sum.golang.org: `h1:MtZvf...`).
- The raw SHA256 of the zip file in the module cache (shown above) is different from the canonical `h1:` checksum. The Go checksum protocol canonicalizes module content before hashing; therefore the raw zip SHA256 will not necessarily match the `h1:` value. I include the raw hash above only for completeness and to document the exact bytes present in my cache.

If you would like any other artifacts (full `go env` output, the exact `.info` and `.mod` files from the cache, or a `git ls-tree` of the `v0.10.0` tag), I can attach them.

```
