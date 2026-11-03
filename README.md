# S3Feather

An S3 browser that runs entirely in your web browser. No server, no install, no build step - one HTML file, open it and start browsing. Works on desktop and mobile, on any platform.

Live: [try it now!](https://wespeakenglish.github.io/S3Feather/) 

![S3Exp browsing a bucket](s3exp.png)

---

## Features

- **The `s3://` address bar** — breadcrumbs when idle, an editable key prefix when focused. Type any `bucket/prefix/` and press Enter to jump straight there.
- **Command palette** — `⌘K` / `Ctrl K` to jump to a prefix, switch bucket, or run any command
- **Keyboard-first** — move, open, select, upload, filter and delete without touching the mouse
- **Multi-bucket** — save several connections and switch with one click
- **S3-compatible** — AWS S3, MinIO, Cloudflare R2, Wasabi, DigitalOcean Spaces, Backblaze B2, Storj, and any S3-compatible endpoint
- **File operations** — upload with per-file progress, download, delete one or many with progress you can stop
- **Zip downloads** — selecting more than one object, or a prefix, downloads a single archive; there is no queue of separate downloads to babysit
- **List and grid views** — grid shows real image thumbnails
- **Presigned share links** — download URLs from 5 minutes to 1 year, or *Never*; 7 days by default
- **Drag and drop** — drop files anywhere on the window to upload into the current prefix
- **Flatten** — one keypress lists every key beneath the current prefix, nested folders included, so you can search a whole subtree instead of walking it
- **Sort and filter** — by name, size or date; live filter across every page loaded for the prefix, and across every level when the listing is flattened
- **Paged listings** — prefixes larger than S3's 1,000-entry page load on demand; sorting and filtering cover every page loaded, and drawing stays capped so a 25,000-object prefix does not freeze the tab
- **Encrypted credential files** — save your connections to a passphrase-protected `.s3vault` file and load them on any machine
- **Nothing stored in the browser** — credentials live in `sessionStorage` for the tab; `localStorage` is never touched
- **No build step** — one file of plain HTML, CSS and JavaScript; nothing to compile or install

---

## Quick start

1. Open `index.html` in any modern browser
2. Click **Add connection** in the sidebar
3. Enter your access key, secret and bucket name, then click **Connect**

Already have a `.s3vault` file from an earlier session? Click **Load from file** instead.

> **Serve over HTTPS if you host it.** Opening the file straight off disk (`file://`) or from `localhost` is fine — browsers count both as secure contexts, so the encrypted save and load work there. On a plain-`http://` origin served to others, `crypto.subtle` may be withheld and those features switch off; everything else still works. The sidebar says which situation you are in, and only calls it a problem when the crypto is actually missing.

---

## The address bar

S3 has no folders. A "folder" is just a set of keys that share a prefix, and the bar at the top is that prefix rather than a pretend directory tree.

- Click any segment to jump up to it
- Click the bar, or press `⌘L` / `Ctrl L`, to edit the raw path — the whole path is selected, so typing replaces it
- Paste a full `s3://bucket/prefix/` URI and it will be parsed
- Name a **different** bucket and S3Exp switches to the matching saved connection. If no connection has that bucket, it says so rather than navigating nowhere.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `⌘K` / `Ctrl K` | Command palette |
| `⌘L` / `Ctrl L` | Edit the address bar |
| `↑` `↓` | Move through objects |
| `Home` / `End` | First / last object |
| `Enter` or `→` | Open a prefix, or download a file |
| `Backspace` or `←` | Up one prefix |
| `Space` | Select the object under the cursor |
| `⌘A` / `Ctrl A` | Select everything in view |
| `/` | Focus the filter |
| `R` | Refresh |
| `M` | Fetch the next 1,000 objects from the bucket, when the listing is partial |
| `[` / `]` | Previous / next page of drawn rows |
| `G` | Toggle list / grid view |
| `H` | Toggle hiding placeholder files |
| `F` | Flatten — list every key below the current prefix, or go back to one level |
| `Delete` | Delete the selection |
| `Escape` | Stop a running upload or delete, then close a dialog, then close the sidebar drawer, then leave the address bar, then clear the filter, then clear the selection |

Typing shortcuts are ignored while a text field has focus. Mouse selection also supports shift-click for a range and `⌘`/`Ctrl`-click to toggle one item.

---

## Flattening a prefix

A normal listing asks S3 for one level: keys at the current prefix, and everything deeper rolled up into folder rows. That is the right default, but it means the filter box only ever searches where you are standing, and the size in the status bar counts only what is on screen.

The tree button beside the eye — or `F`, or the command palette — drops the delimiter from the request, so S3 returns every key beneath the current prefix. Rows then carry the rest of the path in their name: `propustka_Novanta/scan.jpg` sits beside `data.zip`. Typing in the filter now searches the whole subtree, which is how you find a file when you cannot remember which folder it went into.

- **The prefix does not move.** The address bar still says where you are, and **Upload** and **New folder** still act there. Flattening widens what the listing reaches; it is not navigation.
- **The footer changes to match** — `6 objects · 3.8 MB · all levels below sync0/` — because a prefix count means nothing when there are no folder rows.
- **Paging is unchanged.** Still 1,000 keys per request, so a large subtree arrives over several **load next 1,000** clicks; the row cap and `[` / `]` work as they always do.
- **Switching clears the filter and re-lists**, since it changes what is asked of S3 rather than how the answer is drawn. It is per session and resets when you disconnect.

---

## Downloading as a zip

**Download** in the selection bar saves one file as itself and anything more as one archive — the button says which, and shows a zip icon once the selection needs one. There is no separate queue mode: firing a download per object meant browsers throttling them, prompting about them, eventually refusing them, and no way to keep a folder structure. Folder rows have their own zip button, which walks the prefix recursively — everything underneath, nested folders included.

Entries are **stored, not deflated**. Bucket contents are overwhelmingly already-compressed bytes, so deflating would spend CPU to save nothing; it also means there is no compressor to ship, which keeps the whole feature inside the one file. Archives carry UTF-8 entry names and switch to **ZIP64** framing automatically when an object, the archive, or an entry's offset passes 4 GB.

Where the archive is built depends on what the browser offers, and the dialog names the path it took:

| Path | Where | Limit |
|---|---|---|
| Streaming to disk | Chrome, Edge and other Chromium on desktop | Free disk space |
| Browser storage (OPFS) | Chrome and Firefox on Android, plus any browser with OPFS write streams | Storage quota |
| In memory | Everything else, including iOS Safari | 512 MB, refused above that with an explanation |

The first path asks where to save before anything is fetched, then pipes each object straight into that file, so peak memory is one chunk regardless of archive size. The second keeps the archive off the tab's heap by staging it in origin-private storage and saving when it is complete; the staging file is removed afterwards. The third holds everything in memory and is capped, because iOS Safari kills a tab well before a desktop browser would.

**On mobile the first path does not exist** — no mobile browser implements the save picker. Phones get OPFS where available and memory otherwise, so a multi-gigabyte archive is a desktop job.

Other behaviour worth knowing:

- **Progress is reported in bytes, not just files.** The dialog shows objects done, bytes done against the total, the current rate, and which of the three paths is in use — so one slow 2 GB object is visibly downloading rather than looking frozen.
- **It keeps running in a background tab.** Progress is painted between items via animation frames, which a hidden tab stops firing, so the wait is raced against a timer. Without that, switching away mid-archive parked the run until you looked back.
- **Objects are streamed, never buffered whole.** Each one is fetched through a presigned URL and piped into the archive a chunk at a time, so a single 4 GB key costs no more memory than a small one.
- **Stop lands between objects.** An entry is either complete in the archive or absent from it, and stopping discards the partial archive rather than leaving a file that looks openable.
- **Empty-prefix markers are skipped.** Keys ending in `/` are not files; unzippers create the directories they need from the entry names.
- **Names are relative to where you are standing**, so zipping a folder preserves it as a folder inside the archive. Duplicate names get ` (2)` appended, and any path segment that could escape the extraction directory is dropped.

---

## Narrow screens

Below 900px the sidebar stops being a column and becomes a drawer over the listing:

- The **☰** button in the header opens it; the same button, a tap on the dimmed area beside it, or `Escape` closes it
- Picking a connection closes it for you, since the point of opening it was to reach the listing behind it
- Opening a dialog or the command palette closes it too, so nothing stacks on top of it

The listing sheds columns as space runs out rather than squeezing them: the filter box moves to its own row and **Modified** drops below 760px, **Size** below 560px, and the endpoint leaves the status bar below 480px. Row actions and grid checkboxes are hover-revealed on a pointer device and always visible on a touch one.

---

## Saving credentials to an encrypted file

**Save encrypted to file** writes your connections to a `.s3vault` file:

- **AES-256-GCM**, with the key derived from your passphrase by **PBKDF2-HMAC-SHA256 at 310,000 iterations**
- A random 16-byte salt and a fresh random IV on every save
- The key-derivation header is authenticated as GCM additional data, so editing the stored salt or lowering the iteration count causes a decryption failure rather than a quietly weakened key
- Keys, secrets, bucket names, labels and endpoints all sit **inside** the ciphertext — the file leaks nothing about what it holds
- Minimum passphrase length is 10 characters. The passphrase is never stored and **there is no recovery** — lose it and the file is scrap.

**Load from file** decrypts in memory and either adds the file's connections to your current list (skipping exact duplicates) or replaces the list. A wrong passphrase or an edited file fails to decrypt rather than loading anything.

The file gets its own passphrase, independent of anything else, so it is safe to keep as a backup or carry between machines.

---

## CORS configuration

A browser cannot read a bucket that does not allow it. In the AWS console under **S3 → your bucket → Permissions → Cross-origin resource sharing (CORS)**:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

Once it works, narrow `AllowedOrigins` to the exact origin you serve the file from.

---

## Minimum IAM permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    }
  ]
}
```

---

## Connections

Each saved connection stores:

| Field | Description |
|---|---|
| Name | A friendly label, e.g. "Production". Defaults to the bucket name. |
| Access key ID | Your access key |
| Secret access key | Your secret key |
| Bucket | The bucket to browse |
| Region | e.g. `us-east-1`. Leave blank for `us-east-1`. |
| Endpoint | Optional — set it for non-AWS providers |

Connections last for the browser session. Click any entry in the sidebar to switch buckets, or click **✕** to remove it — the ✕ appears on hover, and stays visible on touch devices where there is no hover to rely on. Removing the active connection falls back to the next one.

Each row shows the name on top and the endpoint host (or the region) underneath. When the name is just the bucket name — the default — the bucket is not repeated below it; a name you chose yourself shows `bucket · host`. Anything too long for the row is truncated with an ellipsis, and the full text is in the row's tooltip.

---

## Thumbrella thumbnails (optional)

Optional thumbnails can be generated and cached by setting a Thumbrella
connection string. This is usually a url to a self hosted server or an account
token for the free Cloud service. Leave empty to ignore thumbnails. Read more
at https://thumbrella.dev

---

## S3-compatible services

| Service | Endpoint format |
|---|---|
| Cloudflare R2 | `https://<account-id>.r2.cloudflarestorage.com` |
| MinIO | `http://your-minio-host:9000` |
| Wasabi | `https://s3.<region>.wasabisys.com` |
| DigitalOcean Spaces | `https://<region>.digitaloceanspaces.com` |
| Backblaze B2 | `https://s3.<region>.backblazeb2.com` |
| Storj | `https://gateway.storjshare.io` |

Enter the URL in the **Endpoint** field. Path-style requests and Signature V4 are used automatically. The one exception is a share link longer than 7 days, which is signed with Signature V2 — see *Share links longer than 7 days* below.

> **Mixed content:** if you serve S3Exp over HTTPS, the endpoint must be HTTPS too. A browser blocks `http://` requests from an `https://` page, so a plain-HTTP MinIO host only works when S3Exp itself is on `http://localhost`.

---

## Limits and behaviour worth knowing

- **Large prefixes arrive a page at a time.** S3 answers a listing with at most 1,000 entries, so a bigger prefix loads in pages. The status bar shows *Partial listing — 2,001 loaded* with a **load next 1,000** link beside it, and the same action sits at the foot of the list, in the command palette, and on the `M` key — the status bar is the one part always on screen, so you never have to scroll a thousand rows to find the control. Each click costs the same no matter how much is already loaded.
- **At most 2,000 rows are drawn at once, and you page through the rest.** There is no virtual scroller here — every row is real DOM — and drawing 25,000 of them means a 22 MB HTML string and a list a kilometre tall, where the layout pass alone takes tens of seconds and every sort freezes the tab. So the drawn window is capped and the footer moves it: *Rows 2,001–4,000 of 8,999 · page 2 of 5*, with `[` and `]` on the keyboard. Turning a page is instant, since those rows are already in hand.
- **Two kinds of paging, deliberately worded apart.** **←** and **→** move through rows already loaded; **Fetch next 1,000 from the bucket** is a request to S3. The status bar counts everything loaded, not just what is on screen.
- **The filter and the sort see every loaded page**, not just the drawn window, and the window is sliced after sorting — so sorting by date and reading page 1 is a real answer, and filtering for a key at position 24,999 finds it. Changing the sort or the filter returns you to page 1, because the old page number would no longer mean anything.
- **Select all covers the drawn rows**, not the whole prefix. Deleting a *folder* is different: that enumerates every page itself, so it does not miss anything.
- **Deleting many objects shows progress and can be stopped.** A delete of more than one object, or of any folder, opens a progress dialog with two phases: walking the prefixes to find what is there — counting up, since no total is known yet, and nothing has been deleted at that point — and then the batched deletes, which fill a bar in steps of 1,000 and name the batch in flight (*Batch 3 of 9*). The dialog yields a frame between steps, so the progress is actually painted rather than collapsing into a single jump at the end on a fast connection. **Stop** takes effect at the next batch boundary; the dialog then reports how many were removed, because those are gone and cannot be brought back. A single object still deletes with just a toast.
- **Deleting a folder deletes everything inside it.** Removing `logs/` recursively deletes every object stored under that prefix, including nested folders — the same as Cyberduck, Transmit, or the AWS Console. This can mean a lot of objects behind one confirmation, so double-check the prefix before confirming.
- **Placeholder files are hidden by default.** Zero-byte marker files that other tools drop into a prefix to keep it visible — `.file_placeholder`, `.keep`, `.gitkeep`, `.s3keep`, `.bzEmpty`, `.placeholder`, `.dir`, `.emptyfolderplaceholder` — are hidden from the listing automatically. **Directory markers count too:** any zero-byte key ending in `/` is the object a console or SDK writes to hold an empty folder open, and it comes back from S3 both as that object and as a prefix, so left alone it draws twice — once as the folder row you can open, once as a phantom file row named `sub/` that opens nothing. They still exist in the bucket; toggle them back on with the eye icon in the toolbar, the `H` key, the command palette, or the link in the status bar. Object counts and total size in the status bar reflect only what's currently visible.
- **The eye only governs marker files.** If a prefix has none, revealing them changes nothing on screen, so the button says so rather than looking broken. It is not a way to see objects nested in subfolders — that is what flattening is for.
- **New folder** writes a zero-byte object whose key ends in `/`, which is how a prefix becomes visible in listings. That marker is itself a placeholder, so it stays hidden while the folder row it creates does not.
- **Cancelling an upload** stops the queue; it does not undo transfers that already finished. Files uploaded before you clicked **Cancel** stay in the bucket, a file interrupted mid-transfer is not written, and the dialog reports how many of the batch made it.
- **More than one object always means an archive.** A multi-object selection, or any prefix, goes through the zip path; only a single file is fetched directly. The browser therefore never sees a burst of downloads to prompt about.
- **A single file saves under its last path segment.** A key can carry a path — in a flattened listing every name does — and a slash is not legal in a download filename, so `sub/deep/scan.jpg` saves as `scan.jpg`. Inside a zip the full relative path is kept.
- **Link lifetimes:** download links are signed for 60 seconds, grid thumbnails for 15 minutes, and share links for whatever you pick in the dialog — 5 minutes, 1 hour, 24 hours, **7 days (default)**, 30 days, 90 days, 180 days, 1 year, or Never.
- **Share links longer than 7 days use Signature V2.** Signature V4 caps a presigned URL at 7 days by design — the SDK refuses to sign anything longer and S3 rejects it — so 7 days and shorter are signed with V4, and the longer options fall back to legacy V2, which takes an absolute expiry date. The dialog shows a note whenever a V2 option is picked. Whether the link works depends on the service:
  - **AWS S3** rejects V2 in regions launched after 2014 and for buckets created after June 2020, so on a modern AWS bucket the long options will most likely fail. For long-lived access there, use a public object or CloudFront signed URLs instead.
  - **MinIO, Ceph** and most S3-compatible stores accept V2. **Storj**'s hosted gateway appears to accept it too, but Storj does not document it — open one long link in a private window before relying on it. Storj's own Linksharing Service is an alternative for links that never expire.
- **"Never" means 19 Jan 2038, 03:14:07 UTC.** That is the largest Unix timestamp that fits in a signed 32-bit integer; a later V2 expiry could overflow on servers that parse it that way and be rejected as already expired.
- **Every share link dies with its key.** Whatever expiry was chosen, a link stops working when the access key that signed it is rotated or disabled — and if the connection uses temporary (STS) credentials, when those expire.

---

## Security notes

- Credentials are held **only in `sessionStorage`**, never in `localStorage`, cookies, or on disk.
- `sessionStorage` is scoped to the tab. **Closing the tab clears it; reloading the page does not** — the connections come back after a refresh, which is deliberate. Use **Clear session** to wipe them immediately.
- Presigned URLs are generated locally by the AWS SDK and pass through no third-party server.
- A presigned URL is a bearer token: anyone holding it can download the object, and it cannot be revoked on its own short of rotating the signing key. Prefer the shortest expiry that does the job, and treat 1-year and *Never* links as effectively public.
- Scope credentials to a single bucket with the four actions listed above, and rotate them.

Encryption at rest is not the same as safety while running:

- While you are connected, the plaintext keys are in page memory. Anything running in this page can read them — cross-site scripting, a browser extension, or a tampered CDN script. The AWS SDK is loaded from `sdk.amazonaws.com`, which puts that CDN inside your trust boundary.
- **A leaked `.s3vault` file cannot be revoked.** Changing its passphrase does not help, because the old copy still opens with the old one. If a file goes missing, rotate the S3 keys themselves.

---

## Technical details

- **One file** — `index.html` holds the markup, the stylesheet in a `<style>` block, the icon sprite and all logic. Nothing sits beside it. There is no build step and no npm.
- **Two remote dependencies at load time:** the AWS SDK v2 from the official AWS CDN, and the fonts from Google Fonts. The file therefore needs network access the first time it runs; it is self-contained, not offline.
- **Type** — Space Grotesk for the interface, IBM Plex Mono for anything that came out of S3: keys, sizes, dates, regions and URIs.
- **Icons** — an inline SVG sprite, so no emoji and no image requests.
- **Accessibility** — visible keyboard focus, labelled controls, `aria` roles on dialogs and the object list, `aria-expanded` on the drawer toggle, a live region on the paging footer, and `prefers-reduced-motion` respected.

---

## Browser support

Any modern browser. The encrypted-file features need `crypto.subtle`, which browsers expose only in a secure context, so use HTTPS or `localhost` for those; on other origins they are disabled and the sidebar explains why. JavaScript must be enabled.

---

## License

MIT — do whatever you want with it. Provided as is, with no warranty; see [LICENSE](LICENSE) for the full text.
