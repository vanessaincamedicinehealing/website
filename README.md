# Website

Source files for incamedicine.com.au. It is a plain static site (`index.html`, `styles.css`, `script.js`, `assets/`) with no build step. Open `index.html` in a browser, or run `python3 -m http.server` in this folder, to preview it.

## Deploying to GoDaddy

Every push to `main` runs `.github/workflows/deploy-godaddy.yml`, which uploads the site to GoDaddy cPanel hosting over FTPS. Only files that changed are uploaded. The full-size PNG originals in `assets/` are not uploaded; the page uses the smaller WebP copies.

One-time setup:

1. In GoDaddy, open cPanel for the hosting plan and go to **FTP Accounts**. Use the main cPanel login, or create an FTP account whose directory is the home folder (not `public_html`). Note the FTP server name shown there (often `ftp.incamedicine.com.au` or the server's `secureserver.net` hostname).
2. In GitHub, open the repo's **Settings → Secrets and variables → Actions** and add three repository secrets:
   - `FTP_SERVER`: the FTP server name
   - `FTP_USERNAME`: the FTP username
   - `FTP_PASSWORD`: its password
3. Optional repository variables on the same page:
   - `FTP_SERVER_DIR`: the target folder, default `public_html/`. If the domain is an add-on domain, use its folder, for example `public_html/incamedicine.com.au/`.
   - `FTP_PROTOCOL`: default `ftps`. Set it to `ftp` only if GoDaddy rejects FTPS.
4. Go to **Actions → Deploy to GoDaddy → Run workflow** to do the first upload, then visit the site.

Until the secrets exist, the workflow runs, skips the upload and passes.

`.htaccess` sets up HTTPS redirects, caching, compression and the WebP file type on GoDaddy's Apache servers. Turn on the free SSL certificate in cPanel (**SSL/TLS Status**) before relying on the HTTPS redirect.
