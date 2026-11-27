# Website

Source files for incamedicine.com.au. It is a plain static site (`index.html`, `styles.css`, `script.js`, `assets/`) with no build step. Open `index.html` in a browser, or run `python3 -m http.server` in this folder, to preview it.

## Deploying to GoDaddy

Every push to `main` runs `.github/workflows/deploy-godaddy.yml`, which uploads the site to GoDaddy cPanel hosting over FTPS. Only files that changed are uploaded. The full-size PNG originals in `assets/` are not uploaded; the page uses the smaller WebP copies.

One-time setup:

1. In cPanel, open **SSL/TLS Status** and make sure the free certificate is active for the domain. Do this first: `.htaccess` redirects every visitor to HTTPS.
2. In GoDaddy, open cPanel for the hosting plan and go to **FTP Accounts**. Use the main cPanel login, or create an FTP account whose directory is the home folder (not `public_html`). Note the server hostname cPanel shows for FTP (usually something ending in `secureserver.net`). Use that rather than `ftp.incamedicine.com.au`, because the FTPS certificate is issued for the server's own hostname.
3. In GitHub, open the repo's **Settings → Secrets and variables → Actions** and add three repository secrets:
   - `FTP_SERVER`: the FTP server name
   - `FTP_USERNAME`: the FTP username
   - `FTP_PASSWORD`: its password
4. Optional repository variables on the same page:
   - `FTP_SERVER_DIR`: the target folder, default `public_html/`. If the domain is an add-on domain, use its folder, for example `public_html/incamedicine.com.au/`.
   - `FTP_PROTOCOL`: default `ftps`. Set it to `ftp` only if the first run fails with a TLS or certificate error.
5. Go to **Actions → Deploy to GoDaddy → Run workflow** to do the first upload, then visit the site.

Until the secrets exist, the workflow runs, skips the upload and passes.

`.htaccess` sets up HTTPS redirects, caching, compression and the WebP file type on GoDaddy's Apache servers.

## Running it as a cPanel Node.js app

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
5. To update automatically from GitHub, set the repository variable `FTP_SERVER_DIR` to `incamedicine/` and add the FTP secrets as described above. Content changes show up straight away. Restart the app in cPanel only if `server.js` changes.

An "errno 2" (ENOENT) error in cPanel means a file it expects is missing from the application root: usually the startup file, or `package.json` if you clicked **Run NPM Install**.
