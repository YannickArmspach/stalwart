# TUTO — Stalwart mail on ynk.one: actions only you can do

The agent deploys and verifies everything on the Temps side. The steps below need **your**
accounts (Cloudflare, Namecheap, Hetzner) or **your** secrets (passwords, API token) and can't
be done for you. Do them in order; step 5 tells you when to hand back to the agent.

Server: Temps @ `console.ynk.one` — Hetzner Cloud `91.98.91.115` — mail hostname: `mail.ynk.one`.

---

## 1. Cloudflare — add the ynk.one zone

1. https://dash.cloudflare.com → **Add a domain** → `ynk.one` → **Free** plan.
2. Cloudflare scans and imports existing records. Review them and make sure the list ends up as:

   | Type | Name | Content | Proxy |
   |------|------|---------|-------|
   | A | `console` | 91.98.91.115 | **DNS only** (grey cloud) |
   | A | `dev` | 91.98.91.115 | **DNS only** |
   | A | `*.dev` | 91.98.91.115 | **DNS only** |
   | A | `meridian` | 91.98.91.115 | **DNS only** |
   | A | `mail` | **91.98.91.115** ← replace the old `46.202.129.219` | **DNS only** |

3. **Delete** any imported MX/TXT records mentioning `eforward` / `efwd.registrar-servers.com`
   (that's the old Namecheap forwarding — it's being replaced by your own server).
4. Do **not** create MX/SPF/DKIM/DMARC yourself — Stalwart will publish them automatically (step 7).
5. Note the **two Cloudflare nameservers** shown on the zone overview (e.g. `xxx.ns.cloudflare.com`).

> Everything mail-related must stay **DNS only** (grey cloud). Never enable the orange proxy on
> `mail.ynk.one` — mail protocols can't go through Cloudflare's proxy.

## 2. Cloudflare — API token

1. https://dash.cloudflare.com/profile/api-tokens → **Create Token** → template *Edit zone DNS*.
2. Permissions: `Zone → DNS → Edit` **and** `Zone → Zone → Read`. Zone Resources: *Specific zone* → `ynk.one`.
3. Save the token somewhere safe — you'll paste it **twice**: into the Temps DNS provider (step 6)
   and into the Stalwart webadmin (step 7).

## 3. Namecheap — switch nameservers

1. namecheap.com → Domain List → `ynk.one` → **Nameservers** → *Custom DNS* → enter the two
   Cloudflare nameservers from step 1.5 → save.
2. In **Advanced DNS**, if an *Email Forwarding* section is active, remove the forwarders
   (they stop working anyway once the nameservers change).
3. Propagation takes minutes to a few hours. Check with: `dig NS ynk.one +short` →
   should show `*.ns.cloudflare.com`.

## 4. Hetzner Cloud console

1. https://console.hetzner.cloud → your server → **Networking** → Public network → IPv4
   `91.98.91.115` → **Edit Reverse DNS** → set to `mail.ynk.one`.
2. If a Cloud **Firewall** is attached to the server: allow inbound TCP **25, 465, 587, 143, 993, 4190**.
   (No firewall attached → nothing to do.)
3. Outbound port 25 is **blocked by default** on Hetzner Cloud. Request the unblock:
   Cloud Console → left menu **Limits** (or a support ticket) → request "mail ports / SMTP" removal.
   Reason to give: *self-hosted Stalwart mail server for my own domain ynk.one, rDNS set, SPF/DKIM/DMARC
   configured*. Usually granted after the first paid invoice.
   ⚠️ Until granted: **receiving mail works, sending to other providers doesn't** (messages queue).

## 5. ✋ Hand back to the agent

Once `dig NS ynk.one +short` shows Cloudflare nameservers, tell the agent to continue.
It will attach `mail.ynk.one` to the project, get the HTTPS certificate, verify the webadmin,
and give you the one-time bootstrap admin password for step 7.

## 6. Temps — refresh the Cloudflare DNS provider token

The Cloudflare provider stored in Temps has a dead token, and after leaving Namecheap DNS it becomes
the only way to renew the `*.dev.ynk.one` wildcard certificate. Run this yourself (don't paste the
token in the chat):

```bash
bunx @temps-sdk/cli@0.1.36 --target-context console-ynk-one \
  dns-provider update --id 2 --api-key <YOUR_CLOUDFLARE_TOKEN>
```

Then verify: `bunx @temps-sdk/cli@0.1.36 --target-context console-ynk-one dns-provider test --id 2`

## 7. Stalwart webadmin — first login and mail domain

1. Open `https://mail.ynk.one` → login `admin` + the bootstrap password from the agent.
2. **Immediately change the admin password** (Settings → Administrators).
3. Set the server hostname: `mail.ynk.one` (Settings → Server → Network, if not already set).
4. TLS / ACME (Settings → Server → TLS → ACME): provider *Let's Encrypt*, challenge **DNS-01**,
   DNS provider **Cloudflare**, paste your API token. Stalwart then issues/renews the mail
   certificate itself.
5. DNS management: choose **Automatic** with provider **Cloudflare** + the same token — Stalwart
   publishes MX, SPF, DKIM, DMARC, MTA-STS and autoconfig records for you.
6. **Create the domain**: Management → Directory → Domains → add `ynk.one`.
   (If you skipped automatic DNS: the domain page lists every DNS record to copy into Cloudflare.)
7. **Create your account**: Directory → Accounts → add — name `yannick`, email `yannick@ynk.one`,
   set a strong password. Add aliases `postmaster@ynk.one` and `abuse@ynk.one` pointing to it.
8. Tell the agent — it runs the final verification pass.

## 8. Mail client

Autoconfig should prefill everything once DNS is live. Manual values:

| | |
|---|---|
| IMAP | `mail.ynk.one`, port **993**, SSL/TLS |
| SMTP | `mail.ynk.one`, port **465**, SSL/TLS |
| Login | `yannick@ynk.one` + your mailbox password |

## 9. Final test

- Send a mail from Gmail (or any external account) to `yannick@ynk.one` → it should arrive.
- Reply back → works only after Hetzner unblocks outbound port 25 (step 4.3).
- Optional: send a mail to the address shown at https://www.mail-tester.com — aim for ≥ 9/10.
