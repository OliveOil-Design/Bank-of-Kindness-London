# Kind Words, London

**A QR-sticker kindness wall.** Little weatherproof stickers go up around the city, each carrying a QR code. Scan one and you land on a simple page: read a random kind message left by a stranger, and leave one for the next person. Every message is checked by an AI safety-filter before it goes public, and anything the filter is unsure about waits for a human to review it.

**Live site:** [london.thebankofkindness.com](https://london.thebankofkindness.com/)

---

## What is this, who is it for, and why does it exist?

Cities are full of people having quiet bad days. This project puts a small, free dose of kindness on street furniture: a sticker on a lamppost or bus stop that says invites you to scan it and discover a kind word from a random citizen*

It was born at a [GoodEye](https://goodeye.world) session and built by [Oliver Lloyd](https://www.oliverlloyddesign.co.uk/). Now it's yours too.

This project is **open source** and designed to be copied. If you want a kindness wall in your city; Manchester, Berlin, Lagos, anywhere, you don't need to be a programmer. This guide walks you through the whole thing, step by step, in about an hour. You'll create a few free accounts, copy some text into the right boxes, and end up with your own working wall.

No coding experience is assumed anywhere below. If a step says "open the file," that just means double-clicking it or opening it in a free text editor (I recommend [VS Code](https://code.visualstudio.com/) — free, and works on Mac and Windows).

---

## Join the Bank of Kindness

You have two choices, and both are fine:

1. **Run it entirely independently.** Clone it, rename it, host it wherever you like. It's your wall. (See [License](#license-free-to-share-remix-and-reuse) below.)
2. **Keep it under the Bank of Kindness umbrella.** Your site becomes `yourcity.thebankofkindness.com` — same as our `london.thebankofkindness.com` — so the family of kindness walls grows together and people can see their city is part of something bigger.

For option 2, just email the team at GoodEye:

> **hello@goodeye.world**

Tell them your city and they'll set up your custom subdomain (`yourcity.thebankofkindness.com`) and point it at your site. This is covered in [Step 7](#step-7--get-your-city-domain) below.

---

## Before you start

You'll need these free accounts. Creating them takes about 10 minutes total.

- ✅ A **[GitHub](https://github.com/signup)** account — where the code lives and where your site gets hosted.
- ✅ A **[Supabase](https://supabase.com)** account — this stores the kind messages (the database).
- ✅ A **moderation API key** — the service that checks messages for harmful content (basically, don't be a d*ck). The easiest is [OpenAI](https://platform.openai.com/signup); there's a small pay-per-use cost (typically pennies per month at kindness-wall volumes). Alternatives are listed in [Swapping moderation providers](#swapping-moderation-providers).
- ✅ A **terminal** — a text window where you type commands.


---

## The step-by-step guide

### Step 1 — Get the files

Go to **[github.com/oliveoil-design/Bank-of-Kindness-London](https://github.com/oliveoil-design/Bank-of-Kindness-London)**.

1. Click the green **`<> Code`** button.
2. Click **Download ZIP**.
3. Find the downloaded file (usually in Downloads), and unzip it — on Mac, double-click it; on Windows, right-click → *Extract All*.

You now have a folder containing everything the project needs:

```
kind-words/
├── index.html                          ← the web page itself (you'll edit 3 lines in Step 5)
├── CNAME                               ← your web address (you'll edit in Step 7)
└── supabase/
    ├── schema.sql                      ← builds the database (paste in Step 3)
    └── functions/submit/index.ts       ← the safety-checker (deployed in Step 4)
```


### Step 2 — Create your Supabase backend

Supabase is a free-to-start service that stores your messages safely.

1. Go to [supabase.com](https://supabase.com) and sign up / log in.
2. Click **New project**. Give it a name like `kindness-yourcity`. Choose a password (save it somewhere) and a region near you. Creating it takes a couple of minutes.
3. When it's ready, go to **Settings → API** in the left sidebar (click the ⚙️ gear icon).
4. You'll see three things on that page. Leave the browser tab open — you'll copy them shortly:
   - **Project URL** — looks like `https://abcd1234.supabase.co`
   - **anon public** key — a long string starting with `sb_publishable_...` or `eyJ...`
   - **service_role** key — a secret one. ⚠️ **Never share this or put it in the web page.** It's only used in Step 4, inside Supabase's own servers.


### Step 3 — Create the database

One paste-and-click builds the whole thing: the message table, the security rules, a function that serves a random kind message, and seven starter messages so the wall isn't empty on day one.

1. In your Supabase project, click **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Open the file `supabase/schema.sql` from your downloaded folder in a text editor, select all of it (Cmd/Ctrl+A), copy (Cmd/Ctrl+C), and paste it into the SQL Editor box.
4. Click **Run**. After a few seconds you'll see "Success."


### Step 4 — Deploy the safety-checker function

This is **the most technical step in the whole guide** — but you're still only copying and pasting. This part runs the AI moderation on Supabase's servers, where nobody can get around it.

**4a. Install the Supabase command-line tool.** Follow the one-liner for your computer from [supabase.com/docs/guides/cli/getting-started](https://supabase.com/docs/guides/cli/getting-started) (Mac users: `brew install supabase/tap/supabase`; Windows users: download the installer from the releases page).

**4b. Open a terminal** in your project folder:
- **Mac:** right-click the folder in Finder → *New Terminal at Folder*.
- **Windows:** open the folder in File Explorer, click the address bar, type `cmd` and press Enter.

**4c. Log in and connect** — paste these one at a time, pressing Enter after each:

```bash
supabase login
```

(This opens a browser to confirm it's you.)

```bash
supabase link --project-ref YOUR-PROJECT-REF
```

Replace `YOUR-PROJECT-REF` with the part of your Project URL between `https://` and `.supabase.co` — e.g. for `https://abcd1234.supabase.co` it's `abcd1234`.

**4d. Give the safety-checker its API key** — paste this as one block (replace `sk-...` with your OpenAI API key, which you create at [platform.openai.com/api-keys](https://platform.openai.com/api-keys)):

```bash
supabase secrets set \
  MODERATION_BASE_URL="https://api.openai.com/v1" \
  MODERATION_API_KEY="sk-..." \
  MODERATION_MODEL="gpt-4o-mini" \
  RATE_LIMIT_SALT="$(openssl rand -hex 16)"
```

(The last line makes a random secret that protects people's privacy. On Windows PowerShell, use `RATE_LIMIT_SALT="$([guid]::NewGuid().ToString('N') + [guid]::NewGuid().ToString('N'))"` instead.)

**4e. Deploy it:**

```bash
supabase functions deploy submit
```

When it says the function was deployed, the hard part is done. 🎉

`SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` are injected automatically — you don't set those.

### Step 5 — Connect the page to your backend

1. Open `index.html` in a text editor.
2. Near the bottom, find these three lines (search for `CONFIG` if you can't see them):

```js
var CONFIG = {
  SUPABASE_URL:  'https://xsranwnbxuinswydssqy.supabase.co',
  SUPABASE_KEY:  'sb_publishable_...',
  SUBMIT_URL:    'https://xsranwnbxuinswydssqy.supabase.co/functions/v1/submit',
};
```

3. Replace all three with **your** values from Step 2:
   - `SUPABASE_URL` → your Project URL.
   - `SUPABASE_KEY` → your **anon public** key (this one is *meant* to be public — the database's security rules protect the data, not key secrecy).
   - `SUBMIT_URL` → your Project URL with `/functions/v1/submit` on the end.
4. Save the file. That's all the editing the page needs.


### Step 6 — Put it online

We'll use **GitHub Pages** — free hosting that lives in the same place as your code.

1. Log in to GitHub and click **New repository** (the + icon, top right).
2. Name it something like `kindness-yourcity`, set it to **Public**, and click **Create repository**.
3. On the new repo page, click **uploading an existing file**, then drag in:
   - `index.html` (your edited copy)
   - the `CNAME` file
   - the `assets` and `images` folders and the `london-tube-font` folder (the page needs these to look right)
4. Click **Commit changes**.
5. Go to **Settings → Pages** in the repo, and under *Build and deployment* set Source to **Deploy from a branch**, branch **main**, folder **/(root)**. Click **Save**.
6. Wait a minute or two, then refresh — GitHub shows your site's address, like `https://yourname.github.io/kindness-yourcity/`. **Open it and test it: you should be able to read and leave messages.**


### Step 7 — Get your city domain

To turn `yourname.github.io/kindness-yourcity` into `yourcity.thebankofkindness.com`:

1. Email **hello@goodeye.world** with your city and the URL from Step 6.
2. GoodEye will confirm your subdomain — say it's `manchester.thebankofkindness.com`.
3. Open the `CNAME` file in your text editor, replace its single line with that subdomain, save, and upload the updated file to your GitHub repo (same drag-and-drop as Step 6).
4. Back in **Settings → Pages**, type your subdomain into the *Custom domain* box and save.
5. GoodEye points the domain at your site from their side. DNS can take anywhere from minutes to a day. Once it works, tick **Enforce HTTPS** in the Pages settings.

### Step 8 — Make the QR sticker

The final piece: stickers that link the street to your wall.

1. Point any QR code generator at your live URL — free options include [qr-code-generator.com](https://www.qr-code-generator.com/), [Canva](https://www.canva.com) (search "QR code", can design the sticker in the same place), or [goQR.me](https://goqr.me/). Paste your URL (your city domain, not the github.io one), download the code as PNG or SVG.
2. **Test it with your phone camera before printing anything.**
3. Print tips: print large enough that the code is scannable from ~1 metre; use weatherproof/waterproof sticker paper or a vinyl sticker service; add one line of text next to the code — e.g. *"Scan for a kind word"* — so people know what they're scanning.
4. Stick. Done. 🏙️

📸 *Screenshot: `images/readme/06-qr-sticker.png` — an example of a finished London sticker.*

---

## Running your wall

The rest of this README is the operator's manual, but more detail than a new owner needs on day one.

## Reviewing held messages

If the moderator is ever unreachable or returns something unparseable, the message is stored with `approved = false` so a person can decide. To see the queue:

```sql
select * from public.messages where approved = false order by created_at desc;
-- approve one:
update public.messages set approved = true where id = '...';
-- or remove it:
delete from public.messages where id = '...';
```

## Swapping moderation providers

Change three secrets, redeploy nothing else (`supabase secrets set ...` then it takes effect):

| Provider | `MODERATION_BASE_URL` | `MODERATION_MODEL` (example) |
|---|---|---|
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| GreenPT | *(their base URL)* | *(their model id)* |
| Groq | `https://api.groq.com/openai/v1` | `llama-3.1-8b-instant` |
| Together | `https://api.together.xyz/v1` | *(a chat model id)* |
| Anthropic (OpenAI-compat) | `https://api.anthropic.com/v1` | `claude-sonnet-4-6` |

Any endpoint that accepts `POST /chat/completions` in OpenAI's format works. **Local models** (Ollama / LM Studio at `http://localhost:11434/v1`) only work when the function can reach them — i.e. during local dev with `supabase functions serve`, not from the deployed cloud function, unless you expose them via a tunnel.

## Reporting and rate limiting

Both are built in.

**Report** — every displayed message has a small "Report this" link. Tapping it sends the report through the same `submit` edge function (so it's rate-limited too), and the message is pulled off the wall immediately (it stops being served) and added to the same review queue, where you can restore or delete it:

```sql
select * from public.messages where approved = false order by reports desc, created_at desc;
update public.messages set approved = true, reports = 0 where id = '...';  -- restore
delete from public.messages where id = '...';                              -- remove
```

Reporting hides first and asks questions later — kinder default for a public wall, and false flags are easy to undo.

**Rate limit** — the `submit` function allows **5 submissions and 10 reports per 15 minutes per IP** (tweak the constants at the top of `index.ts`; each has its own counter via the `kind` column in `submission_log`). It keys off a **salted hash of the IP, never the raw IP**, and prunes expired rows on every call — so there's no standing store of who submitted what. Set `RATE_LIMIT_SALT` to any long random string (Step 4 generates one for you).

## A note on privacy

Messages carry no accounts and no logins. A sender may **optionally** leave a first name to sign their note — it's shown publicly on the card as `message — Name`. The name is trimmed to the first word only and capped at 24 characters, server-side, and it's reviewed by the same moderation as the message. Leave it blank and the note stays anonymous. The only other thing resembling personal data is the transient, hashed, auto-pruned rate-limit entry. Keep a one-line visible notice about how messages are handled and reviewed — enough for a street test of user-submitted public content in the UK.

## Security notes (for the curious and the careful)

- The **anon key is meant to be public** — RLS is what protects the data, not key secrecy.
- The **service_role key never leaves the edge function**. Don't put it in `index.html`.
- Visitors can't insert directly; all writes go through moderation.
- **Links are blocked outright** (`http://`, `https://`, `www.`, and common spam-TLD domains) before a moderation call is even spent — a kindness wall has no reason to contain URLs.
- The moderation prompt explicitly treats the message and name as **untrusted data, never instructions**, to blunt prompt-injection attempts; and the rejection "reason" shown to the sender is capped at 200 characters.
- On moderation failure the system **fails safe** (holds for review) rather than publishing.

## How it fits together (technical summary)

- **Read** happens straight from the browser: a plain `fetch` to a Postgres function returns one random *approved* message. Row Level Security means the anon key can only ever read approved rows. (No third-party script is loaded — no supabase-js CDN dependency.)
- **Write** never touches the table directly. The browser posts the text to the `submit` edge function, which moderates it server-side (where the API keys live) and inserts it only if it passes. So no one can bypass moderation, even with the anon key.
- **Reports** also go through the `submit` edge function (with their own rate limit). The anon-facing `report_message` rpc is revoked, so the rate limit can't be bypassed.

---

## License: free to share, remix, and reuse

This project is released under the **[MIT License](LICENSE)** — one of the most open and permissive licenses there is. In plain English:

- ✅ **Use it** for anything, including commercially.
- ✅ **Change it** however you like.
- ✅ **Share it** with anyone.
- ✅ **No permission needed** from us for any of it.

The only requirement: keep the copyright notice in the `LICENSE` file with any copy you distribute.

One friendly, completely non-binding request: if you run a wall in your city, consider linking back to [london.thebankofkindness.com](https://london.thebankofkindness.com/) or this repo — not because you have to, but because the more the family of kindness walls can find each other, the better this gets for everyone.
