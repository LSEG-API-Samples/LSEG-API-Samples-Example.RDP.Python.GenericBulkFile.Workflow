# Agent Instructions

Example project for the LSEG RDP Client File Store (CFS) bulk file workflow. See [README.md](README.md) for background and setup, and [Article.md](Article.md) for the published article.

## Layout

- [RDP-Generic-CFS.ipynb](RDP-Generic-CFS.ipynb): the distributed example notebook. This is the only notebook to edit.
- `RDP-Generic-CFS-Env.ipynb`: personal variant that reads credentials from `.env`. Git-ignored; do not edit or use it as reference.
- [RDP-Generic-CFS.html](RDP-Generic-CFS.html): exported copy of the notebook. Regenerate it, do not hand-edit.
- [postman/](postman/): Postman collection and environment mirroring the notebook steps.
- `cfsvenv/`: local-only and git-ignored. Do not read, search or edit them.

## Run

- Python 3 with a virtual environment, then `pip install -r requirements.txt` and `jupyter lab`.
- The example must work on both Windows and macOS. Do not add platform-specific packages (for example `pywinpty`, `colorama`) to the requirements file.
- `requirements.txt` is a Windows `pip freeze` and `requirements2.txt` lists only direct dependencies (`jupyterlab`, `polling2`, `python-dotenv`, `requests`). Prefer the direct-dependency form for distribution.
- No tests or build step.

## Conventions

- The notebook follows numbered steps 1-7: auth, list packages, list file-sets, get file URL, download, refresh token, revoke token. Keep step numbering consistent with the markdown cells and the Postman collection.
- Plain `requests` calls with `Authorization: Bearer <token>`. Do not introduce wrapper libraries or classes.
- Credentials are placeholders in the notebook cell (`username`, `password`, `clientId`). Never commit real credentials. `.env` is git-ignored; [.env.example](.env.example) lists `RDP_USERNAME`, `RDP_PASSWORD`, `RDP_APP_KEY`.
- Notebook cells are explained by a markdown cell above them. Update both together.
- Download output files (`*.jsonl.gz`, `*.csv`, `*.zip`) are git-ignored.

## Pitfalls

- Step 5 uses `verify=False` as a workaround for a corporate proxy. The notebook warns not to use it in production; keep that warning.
- The S3 URL from step 4 must be used unchanged. Altering it causes a signature mismatch.
- Auth service has rate limits; do not add loops that call the token endpoint repeatedly.
- `acccess_token` (triple c) is initialised in the credentials cell but the code uses `access_token`. Keep this in mind when editing that cell.
- The README run steps contain typos (`CSF`, `activte`, `CFS\bin`); fix them if touching that section.
