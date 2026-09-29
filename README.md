<p align="center">
  <img src="header.png" alt="Abraxas Labs — gitea-public-only-pat-org" width="100%">
</p>

<p align="center">
  <a href="https://abraxaslabs.tech"><strong>abraxaslabs.tech</strong></a>
  &nbsp;·&nbsp;
  <a href="https://github.com/abraxas">github.com/abraxas</a>
  &nbsp;·&nbsp;
  <a href="https://x.com/abraxas_null">@abraxas_null</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/abraxas/gitea-public-only-pat-org">gitea-public-only-pat-org</a>
</p>

# gitea-public-only-pat-org

**Gitea** `1.27.3` — Gitea

Unpublished Gitea source finding: public-only PAT creates a private organization.

| | |
|---|---|
| ID | Unpublished Gitea source finding #4 (no CVE yet) |
| CWE | [CWE-863](https://cwe.mitre.org/data/definitions/863.html) |
| CVSS | **Medium: 6.5** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N` |
| Product | [Gitea](https://github.com/go-gitea/gitea) |
| Affected | all versions **through 1.27.3** (inclusive) |
| Patched | vendor patch — see references |
| Auth | authenticated (see source map) |
| License | [GNU Affero GPL v3.0](LICENSE) |
| Lab | `127.0.0.1` only · vendor/client disclosure pack, not a scanner |

---

## Advisory (from the source map)

routers/api/v1/api.go POST /orgs has no rejectPublicOnly. Sibling POST /user/repos is gated. Org-repo create only checks org visibility.

---

## Entry

- **Method:** `POST`
- **Path:** `/api/v1/orgs`
- **Router:** Authenticated org create. POST /api/v1/orgs has no rejectPublicOnly. POST /api/v1/user/repos does.
- **Notes:** Authenticated unpublished Gitea #4 CWE-863 v1.27.3. Witness: created org visibility=private. Same public-only token cannot create a private user repo (403). Not eval. Not a reverse shell.

### Call chain

- `POST /api/v1/users/{user}/tokens scopes=public-only,write:organization,write:repository,write:user`
- `POST /api/v1/orgs visibility=private → 201`
- `POST /api/v1/user/repos private=true → 403 rejectPublicOnly`

### Lab preconditions

- Gitea 1.27.3
- Public-only PAT that also has write:organization
- DEFAULT_ALLOW_CREATE_ORGANIZATION true (default)

### Witness

POST /api/v1/orgs visibility=private returns 201 and org visibility=private; same token private user-repo create is 403

### Not success

- eval/base64/system payload
- reverse shell
- private user-repo create succeeding
- reading existing private git contents

---

## Patch / remediation

**Do this first:** Apply the vendor patch for **Gitea**. See references.

**Verify after upgrade**

- Re-run `gitea-public-only-pat-org-Abraxas-Labs.py` against the patched build: the mapped witness must **not** appear.
- Confirm the vendor advisory / changeset in the deployed tree (see references).
- A WAF signature is delay, not a patch.

**If you cannot update immediately**

- Disable or isolate the affected component.
- Hunt for the witness condition on production (new privileged users, unexpected files, injected rows — whatever this CVE's map names).

---

## Reproduction (authorized lab)

Target **only** `http://127.0.0.1:8088` (or the loopback you bound). Do not point this script at the internet.

```bash
python3 gitea-public-only-pat-org-Abraxas-Labs.py
```

Success is the **witness** above in the response body. Generic 200 HTML is not it.

---

## Lab images

Loopback stack used to reproduce. Official images unless a `Dockerfile` in this folder builds from source.

- [`lab/docker-compose.yml`](lab/docker-compose.yml)
- [`lab/Dockerfile`](lab/Dockerfile)
- [`lab/run.sh`](lab/run.sh)

```bash
cd lab
docker compose up --force-recreate
```

Bind the vulnerable product tree next to Compose if the YAML mounts a local directory (plugin zip / source tag from the version table). Publish nothing except `127.0.0.1`.

---

## References

- [github.com/go-gitea/gitea](https://github.com/go-gitea/gitea) tag v1.27.3

- Abraxas Labs: [abraxaslabs.tech](https://abraxaslabs.tech) · [github.com/abraxas](https://github.com/abraxas) · [@abraxas_null](https://x.com/abraxas_null)

---

## Records (structured)

```
# Gitea unpublished #4 — public-only PAT creates private org

CWE: CWE-863
Severity: Medium (source review)

## Description

`POST /api/v1/orgs` has no `rejectPublicOnly`. A public-only token with org write scope can create a private organization. The same token cannot create a private user repository.

## Product

Gitea 1.27.3. Lab oracle is visibility=private on the created org, not a shell.
```

---

## License

This disclosure pack is licensed under the **GNU Affero General Public License v3.0**. See [LICENSE](LICENSE).

---

## Disclaimer

This pack is for **the vendor, the site owner, and licensed labs**. The script talks to `127.0.0.1`. Using it against systems you do not own is not authorized by Abraxas Labs. No warranty.

<p align="center">
  <a href="https://abraxaslabs.tech">abraxaslabs.tech</a> ·
  <a href="https://github.com/abraxas">github.com/abraxas</a> ·
  <a href="https://x.com/abraxas_null">@abraxas_null</a>
</p>
