# Skill: GitHub API Curl Standard (Bash)

## Purpose

When interacting with the GitHub REST API from Bash scripts, always use a standardized curl pattern that captures:

* HTTP response headers
* Response body
* HTTP status code
* GitHub Request ID

This ensures proper observability, troubleshooting, and error handling in CI/CD pipelines and automation workflows.

---

## Mandatory Pattern

Always separate:

* Headers (`-D`)
* Response body (`-o`)
* HTTP status code (`-w`)

Never rely on parsing curl stdout directly.

```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  -D /tmp/gh_headers.txt \
  -o /tmp/gh_body.json \
  -w "%{http_code}" \
  https://api.github.com/endpoint > /tmp/gh_status.txt
```

---

## Extract Metadata

After the request completes, extract the GitHub Request ID and HTTP status code.

```bash
REQUEST_ID=$(grep -i "x-github-request-id:" /tmp/gh_headers.txt | cut -d':' -f2- | xargs)
HTTP_CODE=$(cat /tmp/gh_status.txt)
```

The Request ID is critical when opening support cases with GitHub or correlating logs with GitHub backend activity.

---

## Validate HTTP Response

Every API call must validate the returned HTTP status code.

```bash
if [ "$HTTP_CODE" -lt 200 ] || [ "$HTTP_CODE" -ge 300 ]; then
  echo "❌ Error: HTTP $HTTP_CODE"
  echo "GitHub Request ID: $REQUEST_ID"
  cat /tmp/gh_body.json
  exit 1
else
  echo "✅ Request successful"
fi
```

---

## Error Handling Requirements

For non-2xx responses:

1. Print the HTTP status code.
2. Print the GitHub Request ID.
3. Print the response body.
4. Fail the script with a non-zero exit code.

Required output:

```text
❌ Error: HTTP 404
GitHub Request ID: ABCD:1234:5678:EFGH
{
  "message": "Not Found"
}
```

---

## Success Handling Requirements

For successful requests:

```text
✅ Request successful
```

Optionally log:

```bash
echo "GitHub Request ID: $REQUEST_ID"
```

for traceability purposes.

---

## Rationale

This pattern provides:

* Consistent troubleshooting across workflows.
* Visibility into GitHub API failures.
* Easier support engagement with GitHub.
* Separation of response metadata from payload data.
* Predictable behavior in CI/CD environments.

---

## Do Not

Avoid patterns such as:

```bash
RESPONSE=$(curl ...)
```

or

```bash
curl ... | jq ...
```

when HTTP status codes and headers are required for diagnostics.

Always capture headers, body, and status code independently.
