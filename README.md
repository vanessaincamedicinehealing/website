# Website

Source files for incamedicine.com.au. It is a plain static site (`index.html`, `styles.css`, `script.js`, `assets/`) with no build step. Open `index.html` in a browser, or run `python3 -m http.server` in this folder, to preview it.

## Deploying to GoDaddy

The live site runs on GoDaddy **Managed Hosting for WordPress**. The static files sit in the WordPress web root next to WordPress's own files, and the home page is served from `index.html`.

Every push to `main` runs `.github/workflows/deploy-godaddy.yml`, which uploads `index.html`, `styles.css`, `script.js` and `assets/` over SFTP. The full-size PNG originals stay in the repo and aren't uploaded. The workflow never deletes anything and never touches WordPress files or the server's `.htaccess`. It refuses to upload unless the target folder contains both `index.html` and `wp-config.php`.

One-time setup:

1. In GoDaddy, open the site's **Hosting Settings → SSH/SFTP login → Create New Login**. GoDaddy shows the username and password once. Creating a new login disables the old one.
2. Add three repository secrets (GitHub **Settings → Secrets and variables → Actions**, or `gh secret set NAME -R vanessaincamedicinehealing/website`):
   - `SFTP_SERVER`: the SFTP hostname shown in Hosting Settings (ends in `ssh.myftpupload.com`)
   - `SFTP_USERNAME`: the SFTP username
   - `SFTP_PASSWORD`: its password
3. Optional repository variable `SFTP_SERVER_DIR`: the web root. If it isn't set, the workflow tries the login's home folder, then `html/`.
4. Go to **Actions → Deploy to GoDaddy → Run workflow**. With "dry run" ticked (the default), it only connects and lists the web root. Untick it to upload.

Until the secrets exist, the workflow runs, skips the upload and passes.

### Caching: flush after every deploy

GoDaddy's CDN caches every page and file for 31 days and ignores the repo's `.htaccess`. After a deploy, open **Hosting Settings → Flush Cache → Flush Now**, or visitors may keep seeing the old version. The workflow adds `?v=<commit>` to the CSS and JS links, so returning visitors' browsers pick up style and script changes. A browser that has already cached the HTML page itself can keep it for up to 31 days.

`.htaccess` in this repo is for ordinary cPanel hosting only. The workflow doesn't upload it.

## Running it as a cPanel Node.js app (not used by the live site)

The repo also works with cPanel's **Setup Node.js App**. `server.js` is a small static file server with no dependencies, so you don't need **Run NPM Install**. `app.js` just loads `server.js`, so cPanel's default startup file works too. The server only serves `index.html`, `styles.css`, `script.js` and files in `assets/`; anything else returns 404.

1. Put the site files in their own folder in the home directory, for example `incamedicine`. Don't use `public_html` itself, because cPanel writes the app's settings into `public_html/.htaccess` and the site's own `.htaccess` would overwrite them.
2. In **Setup Node.js App**, create or edit the application:
   - Node.js version: 18 or newer
   - Application mode: Production
   - Application root: `incamedicine`
   - Application URL: the domain
   - Application startup file: `app.js` (or `server.js`)
3. Click **Save**, then **Restart**.
4. Turn on HTTPS with **Domains → Force HTTPS Redirect** in cPanel. The `.htaccess` redirect doesn't apply to a Node app.
5. The GitHub deploy workflow targets Managed WordPress, not a Node app, so a Node app has to be updated by hand.

An "errno 2" (ENOENT) error in cPanel means a file it expects is missing from the application root: usually the startup file, or `package.json` if you clicked **Run NPM Install**.
