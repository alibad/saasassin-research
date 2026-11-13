# SaaSassin Research

**Free, open-source Dovetail alternative.** Turn raw customer evidence into durable insight without surrendering the archive.

Research repository for product and research teams handling sensitive customer evidence.

- **Try it:** [research.saasassin.net](https://research.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/dovetail](https://saasassin.net/targets/dovetail)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-research&project-name=saasassin-research&repository-name=saasassin-research&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-research%231-create-a-firebase-project)

Dovetail is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Research is an independent, clean-room product and is not affiliated
with or endorsed by Dovetail.

## What you can do today

| Capability | Status |
|---|---|
| Workspace, invites (admin / researcher / viewer), projects, audit log | ✅ |
| Add interview: upload/paste `.vtt` / `.srt` / `.txt` with live preview, participant and consent (status, captured, retention) | ✅ |
| Transcript as speaker + timestamp segments; correct text and speaker; rename speaker everywhere | ✅ |
| Select text to create a tagged highlight (snaps to words, validated server-side); re-anchors after correction | ✅ |
| Insights must cite evidence; backlinks jump to and mark the exact span; themes group insights | ✅ |
| Tag frequency, rename/merge tags, edit highlight tags/notes | ✅ |
| Project search (server-side, membership- and folder-aware, ranked, deep-linked) | ✅ |
| Redaction with audit; source deletion (owner/admin) with reason | ✅ |
| Legal holds. Place or release with a reason; while held, delete, redact and retention runs are blocked and the attempt is audited | ✅ |
| Retention run. Admins preview a dry run, then apply. It deletes expired or consent-withdrawn sources that are not held, with one audit event per deletion plus a summary | ✅ |
| Folders with access. Restricted folders are visible only to listed members, owners and admins, everywhere: project view, search, project export, insight evidence (with a "N restricted links" notice), versions, reports, retention queue and audit log. Workspace export is limited to owners and admins, who see everything | ✅ |
| Insight version history. Every create/edit is an immutable version (`insightVersions/{id}_vN`) with a word-level diff view, plus evidence added/removed and theme changes | ✅ |
| Reports. Pick insights in order and add a summary; a read-only page at `/apps/research/reports/{tenant}/{project}/{report}` shows each insight with its evidence quotes. Only workspace members can open it; non-members see a "not available" page | ✅ |
| Dovetail CSV/JSON import with mapping preview and report | ✅ |
| Versioned project export and workspace export | ✅ |

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Research is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Research signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` denies all browser access to product data: every read and write goes through the server routes, which check the session, tenant membership and role with the Firebase Admin SDK. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Research at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=research`,
and `src/proxy.ts` maps `/` to the product (`/apps/research`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-research&project-name=saasassin-research&repository-name=saasassin-research&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-research%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Dovetail, running Research for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/dovetail#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
