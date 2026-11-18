# gitpin template

This template gives your files a CID, a name for exact bytes that anyone can verify and that keeps working wherever the bytes are hosted. A CID is the name of the exact bytes of a file: if the bytes change, the CID changes. It is good for datasets you cite, NFT or dapp assets, software releases, ENS sites.

Here "publish" is what gitpin does, and a "pin" is something published from the `pins/` folder in your repo. It is not a copy kept on many machines. Your files are hosted by GitHub Pages, so they are online exactly while your Pages site is; the CID is the lasting part.

If you only want a website, plain GitHub Pages is the better choice and this template is not for you.

## What this does

Put a file in the `pins/` folder, commit, and the [gitpin](https://github.com/ipni/gitpin#readme) action publishes it to [IPFS](https://ipfs.tech/). IPFS is a network where files are found by their name (their CID) instead of by an address. Every file or folder you put in `pins/` gets its own CID, and anyone with that CID can fetch exactly those bytes from the network and verify they match.

This is a good way to get started with IPFS, but on its own it is not resilient. The files live only on GitHub Pages, so a GitHub outage, a deleted repo or a takedown makes them unreachable. To make them resistant to outages and censorship, also pin the CID on other nodes: your own IPFS node, a friend's, or a pinning service (a service that keeps copies of files by CID). Every copy serves the same CID, so your links keep working.

## Set it up

1. On [github.com/ipni/gitpin-template](https://github.com/ipni/gitpin-template) click "Use this template" > "Create a new repository". Name it exactly `<your-username>.github.io` (lowercase is fine) and make it **Public**. Pages on free accounts needs a public repo. An account can have only one such repo. If you already use yours for a website, adding gitpin to it would replace that website, so use a custom domain instead (see [Use a custom domain](https://github.com/ipni/gitpin#use-a-custom-domain)) or create the repo under a GitHub organization you own.
2. Make a key. In a terminal, run `ssh-keygen -t ed25519 -C gitpin -f gitpin-key` and press Enter twice when it asks for a passphrase (gitpin needs a key without one). It makes two files in the current folder: `gitpin-key` (the private key) and `gitpin-key.pub`. Make a new key just for gitpin, never reuse your own SSH login key: any workflow in the repo can read the secret.
3. Add the secret. Open `gitpin-key` in a text editor and copy all of it, including the `BEGIN` and `END` lines. In the new repo: Settings > Secrets and variables > Actions > New repository secret, name `GITPIN_KEY`, paste, Add secret. Then delete both key files; the secret is the only copy you need.
4. Actions tab > "gitpin" workflow > "Run workflow" (or edit any file in `pins/` and commit). The first run creates the `gh-pages` branch, then waits up to 10 minutes for Pages and fails with a message ending in "is not serving the new head yet", because Pages is not serving the branch yet. That is expected.
5. Settings > Pages > Build and deployment > Source "Deploy from a branch", Branch `gh-pages`, folder `/ (root)`, Save. GitHub usually has Pages on already for a repo named `<your-username>.github.io`, serving `main`; the change is only the branch. Saving does not publish anything by itself; the re-run in the next step does.
6. Re-run the workflow. When it is green, the run summary lists each pin with its CID and links; `https://<your-username>.github.io` shows the same list.

The template ships `pins/hello.txt` containing `Hello from my first IPFS pin!`; that is your first pin.

## Add your own files

Put a file or folder in `pins/` (via "Add file" > "Upload files" on GitHub) and commit. Each top-level file or folder in `pins/` is one pin with its own CID.

- Changing a file gives it a new CID, and the old CID's blocks are removed from the site and its record dropped, so an old CID someone cited stops resolving from this site.
- Names starting with `.` (such as `.gitkeep`) are skipped at every level.
- Symlinks make the run fail; use real files.
- A pin that is a folder has one directory CID, and each file inside is reachable by path: `https://inbrowser.link/ipfs/<cid>/<name>` or `ipfs cat <cid>/<name>`.

## Open your pins

- `https://inbrowser.link/ipfs/<cid>` (works in the browser and verifies the content)
- `https://ipfs.io/ipfs/<cid>` and `https://dweb.link/ipfs/<cid>` work in a web browser only (they load an in-browser IPFS client); scripts and `curl` get an error from them, so use Kubo or the CID with an IPFS library instead.
- `ipfs cat <cid>` in a recent Kubo (0.43 or newer was tested; older versions may not fetch over HTTP from a site).

A new pin can take a few minutes, sometimes up to an hour, to show up on cid.contact; until then gateways may not find it. The run then ends with a yellow warning, not a failure.

## Remove a pin

Delete the file or folder from `pins/` and commit. The blocks are removed from the site and cid.contact is told to drop the record.

## Good to know

- Your pins are public. Anyone can fetch them, and the CIDs and your peer ID are listed on cid.contact. Do not publish secrets or personal data.
- This is not durable storage and not censorship resistant on its own: GitHub is the only host. Your files are online while your GitHub Pages site is. Delete the repo or turn off Pages and they are gone (other IPFS nodes may still hold copies they fetched). To keep a copy elsewhere with the same CID, add the CID to a pinning service or run `ipfs pin add <cid>` on your own Kubo node.
- At most 500 MiB and 50,000 blocks of content per repo.
- The key signs your advertisements. Keep it only in the `GITPIN_KEY` secret. If it leaks, make a new key with the `ssh-keygen` command in step 2, replace the secret, and delete `gitpin.json` and the `ipni/` folder from the `gh-pages` branch before the next run; `ipfs/` can stay. On GitHub: open the `gh-pages` branch, open `gitpin.json`, "..." > "Delete file", commit; open the `ipni` folder, "..." > "Delete directory", commit; or do the same in a local clone and push.

## If something goes wrong

- **"... is not serving the new head yet"**: Pages is off, the source is not set to the `gh-pages` branch, the repo is private, the repo is not named `<owner>.github.io` and no `base-url` is set, or a custom domain is not pointing at Pages yet.
- **"... redirects to ..."**: the site redirects, usually because a custom domain is set while `base-url` is still the default `https://<owner>.github.io`, or the domain was given as the apex while Pages serves `www` (or the other way round); set `base-url` to the exact origin that serves the site.
- **"The GITPIN_KEY secret is not set"**: add the secret under Settings > Secrets and variables > Actions.
- **"GITPIN_KEY belongs to ... but gitpin.json was written by ..."**: the key changed; see the key bullet above.
- **"GITPIN_KEY is not an Ed25519 key"**: the secret holds something else, often the `.pub` file or a cut-off paste; paste the whole private key file again, or make a new key.
- **"GITPIN_KEY has a passphrase"**: make a new key and press Enter at the passphrase prompt.
- **"needs a repo named `<owner>.github.io`"**: rename the repo, or use a custom domain (see the [main README](https://github.com/ipni/gitpin#readme)).
- **A yellow "cid.contact has not indexed every pin yet" warning**: wait a few minutes and re-run the workflow.
- **The push to `gh-pages` fails**: the workflow's `actions/checkout` must keep its default credentials; `persist-credentials: false` breaks the push that the `ipni/gitpin@v1` action makes.
