# nodejs-goof (fork of snyk-labs/nodejs-goof) — expected upgrade-impact verdicts

Scan branch `main` @ 87a8853. Lock: `package-lock.json` v2 (so the apply stage regenerates it with npm).
Targets are the highest first-patched version the scanner computes; where the fix crosses majors the
verdict mostly rests on notes + grep, because none of these packages ship their own types (expect the
`@types` fallback or no diff).

| package | installed | fix (advisory data, Oct 2026) | expected | what to look for |
|---|---|---|---|---|
| st | 0.2.4 | 1.2.2 (major) | **NEEDS CHANGES** or SAFE with a caveat | `app.js:72` `st({ path: './public', url: '/public' })`; st 1.x keeps `path`/`url` options, so SAFE is defensible; a wrong answer is NEEDS CHANGES citing an option that still exists |
| lodash | 4.17.4 | 4.18.0 | **SAFE**, medium | `routes/index.js` uses `_.merge`-style helpers; same reasoning as the fixture |
| marked | 0.3.5 | 4.0.10 (major) | **NEEDS CHANGES** | `routes/index.js` calls `marked(...)` as a function; marked 4 is ESM-friendly but still exports `marked()`; 1.0 dropped the `sanitize` option and `marked.setOptions` changed; watch whether the agent finds the actual call |
| moment | 2.15.1 | 2.29.2 | **SAFE**, medium | `moment(...)` formatting only |
| mongoose | 4.2.4 | 9.x (major x5) | **NEEDS CHANGES** | `mongoose-db.js` + `routes/index.js`: `mongoose.connect` options, callbacks removed in 7+, `Model.find` callback style; any confirmed callback-style call site is correct |
| express | 4.12.4 | 4.22.0 | **SAFE**, medium | stays on 4.x; nothing removed |
| ejs | 1.0.0 | 3.1.10 (major) | **NEEDS CHANGES** likely | used through `consolidate`/`ejs-locals`; 2.x removed the `filters`/`open`/`close` options; the agent must find where ejs is configured (`app.js` view engine) |
| adm-zip | 0.4.7 | 0.6.1 | **SAFE**, medium | `routes/index.js:16` `AdmZip` + `extractAllTo`; API kept |
| body-parser | 1.9.0 | 1.20.6 | **SAFE** | `bodyParser.urlencoded/json` only |
| typeorm | ^0.2.24 -> resolved 0.2.x | 0.3.x+ (major) | **NEEDS CHANGES** | `typeorm-db.js`: `createConnection`/`getConnection` were deprecated in 0.3 and removed later; a cited `typeorm-db.js:6`/`:21` is correct |
| validator | ^13.5.2 -> latest 13.x | 13.15.x | **SAFE** | `isEmail`, `isMobilePhone('he-IL')`, `isAscii` all exist |
| dustjs-linkedin | 2.5.0 | 3.0.0 (major) | **UNKNOWN** or NEEDS CHANGES | only `require`d; dust 3 changed the `dust.render` callback signature? the honest answer is "no evidence"; watch for invented sites |
| express-fileupload | 0.0.5 | 1.1.9 (major) | **NEEDS CHANGES** | 1.x changed `req.files` shape (`mv()` promise) — `routes/index.js` import handler |
| jquery | ^2.2.4 | 3.5.0 (major) | **SAFE** for the server | only bundled by browserify for the browser; no server call sites; the agent should say it is not used server-side |
| hbs | ^4.0.4 | no fix | not assessable (400) | |

Traps: `exploits/` and `public/js/bundle.js` contain copies of library code — grep hits there are NOT call sites.
