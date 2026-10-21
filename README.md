# ShipFast site

This is a small, two-page website for **ShipFast**, a concept company that sells route-planning software to delivery and logistics fleets. ShipFast isn't a real business. The site is the front door of an AI sales-automation system, the AI Revenue Engine: someone fills in the "Request a walkthrough" form, the form is sent to a Make scenario, and the visitor lands on a thank-you page while Make handles the lead in the background.

The site is plain HTML and CSS. It has no JavaScript, cookies, analytics, external fonts or images. It asks search engines not to list it (`noindex, nofollow`), but anyone with the link can open it.

## What's in this folder

- `index.html` is the home page: what ShipFast does, how it works, an illustrative example and the walkthrough request form.
- `thanks.html` is the page visitors land on after they send the form.
- `styles.css` holds the styles for both pages. Light or dark mode follows the visitor's device setting.
- `README.md` is this file.

## Preview it on your computer

1. Open a terminal in this folder.
2. Run `python -m http.server 8000`. On Windows, if `python` opens the Microsoft Store, run `py -m http.server 8000` instead.
3. Open <http://localhost:8000> in your browser. The thank-you page is at <http://localhost:8000/thanks.html>.
4. Press Ctrl+C in the terminal to stop.

You can also just double-click `index.html` for a quick look.

**The form can't be sent from the preview, and that's expected.** Until you fill in the placeholders below, the form posts to the preview server, which doesn't accept form posts. You'll see an error such as `501 Unsupported method ('POST')`. After you've filled them in, the form posts straight to Make, and Make sends you to the thank-you page on the live site, not the local one.

## Fill in the two placeholders

`index.html` has two placeholders that you replace with real values before the form will work.

Here's how the two lines look now:

```html
<form method="post" action="__MAKE_W2_WEBHOOK_URL__">
<input type="hidden" name="form_token" value="__FORM_TOKEN__">
```

### Step 1: copy the Make webhook address

In Make, open scenario **W2**, click its first module (the custom webhook) and copy the address. It looks like this (made-up example):

`https://hook.eu2.make.com/x7k2p9q4w1example`

It's the same value as `MAKE_W2_WEBHOOK_URL` in the main project's `.env` file.

### Step 2: get the form token

Use the `FORM_TOKEN` value from the main project's `.env` file. If you haven't made one yet, run this and copy what it prints:

```
python -c "import secrets; print(secrets.token_urlsafe(24))"
```

It prints a random string like this one (made-up example):

`Qm7tR2xVb9LkP4sW1nZc8HfJ`

Use a different value from `APPROVAL_SECRET`. Make W2 must check against the same `FORM_TOKEN` value.

### Step 3: paste them in

Open `index.html` in a text editor such as VS Code. On GitHub you can also open the file and click the pencil icon. Use Find to locate `__MAKE_W2_WEBHOOK_URL__` and replace it with the webhook address. Do the same for `__FORM_TOKEN__` and the token. With the example values above, the two lines become:

```html
<form method="post" action="https://hook.eu2.make.com/x7k2p9q4w1example">
<input type="hidden" name="form_token" value="Qm7tR2xVb9LkP4sW1nZc8HfJ">
```

To check you didn't miss one, search the file for `__` (two underscores). Nothing should be found.

### Is the form token a secret? No.

The site is public, so anyone can see the webhook address and the token by viewing the page source. The token is only a **light bot filter**. It stops random scanners and lazy bots that post to the webhook without loading the page, but anyone who reads the page can copy it.

Make also checks the hidden **honeypot** field, `fax`. People never see or reach it: it sits off-screen, the Tab key skips it, and screen readers ignore it. So a person always leaves it empty, while simple bots fill in every field they find. If `fax` isn't empty, or the token is wrong, Make ignores the submission. The visitor still lands on the thank-you page, so bots can't tell they were caught.

**Never put a real secret in this site**, such as an API key, the HubSpot token or `APPROVAL_SECRET`. If spam starts getting through, make a new `FORM_TOKEN` and change it in both places: this page and Make's check.

## Publish it on GitHub Pages

GitHub Free only publishes Pages sites from **public** repositories. That's why the site has its own public repo, separate from the private main project.

The examples below use the GitHub username `alex-example` and the repo name `shipfast-site`. Swap in your own.

1. On GitHub, click **New repository**. Name it `shipfast-site`, choose **Public**, and click **Create repository**.
2. On the new repo's page, click **uploading an existing file**. Drag in the four files (`index.html`, `thanks.html`, `styles.css`, `README.md`), then click **Commit changes**.
3. Go to the repo's **Settings → Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, then choose the `main` branch and the `/ (root)` folder. Click **Save**.
4. Wait a minute or two (up to 10 minutes the first time), then refresh the page. GitHub shows **Your site is live at** `https://alex-example.github.io/shipfast-site/`.
5. Open that address to check the site. The thank-you page is at `https://alex-example.github.io/shipfast-site/thanks.html`.

Every later commit to `main` republishes the site automatically.

If you prefer git to uploading, first copy the four files into a new, empty folder outside the main project. That way the public repo gets only these files. Then, inside that folder:

```
git init
git add index.html thanks.html styles.css README.md
git commit -m "ShipFast site"
git branch -M main
git remote add origin https://github.com/alex-example/shipfast-site.git
git push -u origin main
```

Then carry on from step 3.

## What Make must send back

When someone sends the form, their browser posts the fields to Make's webhook and waits for an answer. Make W2 must answer with a redirect to the thank-you page:

- **Status:** `303` ("See Other"). This tells the browser to go to another page.
- **Header:** `Location` set to the site address plus `thanks.html`, written out in full, including the repo name. In the example: `https://alex-example.github.io/shipfast-site/thanks.html`.
- **Timing:** within **180 seconds**.

In Make, the form branch of W2 should start with a **Webhooks → Webhook response** module. Set its status to `303`, add a custom header named `Location` with the thank-you address as its value, and leave the body empty. Keep it **before** the slow steps (company lookup, AI decision, HubSpot, email). The visitor is redirected right away, and Make carries on with the lead after that.

If no Webhook response runs within 180 seconds, Make answers by itself with `200 Accepted`. The visitor then sees a plain page with the word "Accepted" instead of the thank-you page. The same thing happens if W2 is switched off. So if you ever see "Accepted", check that W2 is on and that the Webhook response comes first in the form branch.

## What the form sends to Make

The form sends `application/x-www-form-urlencoded` data with exactly these fields. Here's a worked example of one test submission:

| Field | Example value | Notes |
|---|---|---|
| `full_name` | `Test Lead` | Required |
| `work_email` | `test.lead+1@example.com` | Required, must look like an email |
| `company` | `Example Courier Co` | Required |
| `website` | `https://www.example.com` | Required, must be a full web address |
| `fleet_size` | `50-199` | Required. One of `1-9`, `10-49`, `50-199`, `200-999`, `1000+` |
| `company_size` | `50-199` | Required. Employees: one of `1-49`, `50-199`, `200-499`, `500-1000`, `1001+`. Make routes by it (decision #40) |
| `message` | `We run 60 vans.` | Optional, can be empty |
| `form_token` | `Qm7tR2xVb9LkP4sW1nZc8HfJ` | Hidden. Your `FORM_TOKEN` |
| `source` | `shipfast-demo-site` | Hidden, always this value. Make W2 checks it, so it keeps its original name |
| `fax` | *(empty)* | Hidden honeypot. Must be empty |

On the wire, that same submission looks like this:

```
full_name=Test+Lead&work_email=test.lead%2B1%40example.com&company=Example+Courier+Co&website=https%3A%2F%2Fwww.example.com&fleet_size=50-199&message=We+run+60+vans.&form_token=Qm7tR2xVb9LkP4sW1nZc8HfJ&source=shipfast-demo-site&fax=
```

Make decodes it back to the values in the table. For example, `1000%2B` becomes `1000+` again.

The browser checks the fields before sending. Required fields can't be empty, the email and website must be valid, and each field has a maximum length. That only helps honest visitors, though, because anyone can post to the webhook directly. Make must still check everything itself.

## Test the whole flow

1. Open the live site and fill in the form with test details, for example the values from the table above. The company and website can be a real logistics company, because the system only looks up public company facts. The name and email must be test details.
2. Click **Request a walkthrough**. Within a few seconds you should land on `thanks.html`.
3. In Make, open W2's run history and check that the run received the fields above.

## Rules for editing this site

- **Keep the field names and fleet-size values exactly as they are.** Make expects them. If you rename one here, rename it in Make too.
- **Keep the footer line** "ShipFast is a concept product." It keeps visitors from taking ShipFast for a real company. The pages also ask search engines not to list them (`noindex`).
- **Keep the example labeled "Illustrative example".** Its numbers are made up and must never be presented as a real customer's results.
- **Autocomplete is off on purpose,** so browsers don't fill in anyone's saved name, email or employer.
- **Keep it static.** GitHub Pages only serves files. All processing happens in Make.
