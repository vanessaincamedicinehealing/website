# Website

The site at incamedicine.com.au: `index.html`, `styles.css`, `script.js` and the images in `assets/`. There's no build step. To preview it, open `index.html` in a browser.

## Updating the site

1. Edit the files and push to `main`. GitHub uploads them to GoDaddy automatically, which takes about a minute. You can see each run under the repo's **Actions** tab.
2. In GoDaddy, go to **Hosting Settings → Flush Cache → Flush Now**. Skip this and visitors can keep seeing the old version for weeks.

Images: add a `.webp` or `.jpg` to `assets/`. `.png` files aren't uploaded, so keep large PNG originals there as source copies only.

## One-time setup

The site is on GoDaddy Managed WordPress, in the same folder as WordPress's own files. The upload only adds or replaces the site files; it never deletes anything.

1. In GoDaddy, go to **Hosting Settings → SSH/SFTP login → View or Change → Create New Login** and note the username and password.
2. Add them as GitHub repository secrets: in GitHub, **Settings → Secrets and variables → Actions**.
   - `SFTP_USERNAME`: the username
   - `SFTP_PASSWORD`: the password

Until those secrets exist, pushes still succeed but upload nothing.
