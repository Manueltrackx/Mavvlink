# Linkora: build-out review notes

## Changed
- backend/index.ts, PUT /api/site: server-owned fields (plan, subscriptionStatus,
  subscriptionStartedAt, expiresAt, views, customDomain, domainStatus) are now taken
  from the stored record, never from the client. Previously any signed-in user could
  grant themselves Pro or mark a custom domain as approved.

## Verified
- All TS/TSX source files parse (esbuild). tests/tests.json and build_manifest JSON are valid.
- Every frontend /api call matches a backend route.
- Not run: full `vite build` and `tsc` type-check (dependencies could not be installed in this sandbox, no network).

## Open issues (not changed)
1. "Lead capture" is advertised on the Pro plan but has no backend route or UI.
2. Plan limits (Free: 5 links) are not enforced server-side; PUT /api/site slices links to 40.
3. Custom domain (POST /api/domain) is not gated on the Pro plan server-side.
4. GET /api/analytics and GET /api/admin read only the first 100 rows of a table and filter in memory, so counts are wrong once data grows. Needs a userId-filtered query.
5. POST /api/event is unauthenticated and unthrottled, so view/click counts can be inflated.
6. realtime-subscribers.ts routes (realtimeSubscriptionRoutes) are defined but never mounted, and the frontend does not use them. Either wire up or remove.
7. Default plan label is "Starter" in App.tsx but "Free" in the backend and pricing table.
8. tests/tests.json holds manual browser scenarios only; package.json has no test runner.

## Menu button (second pass)
- src/index.css: the header menu button was hidden above 620px, so it was missing on desktop and tablet. It now shows at all widths; the menu panel is also visible at all widths when opened.
- src/App.tsx: the button toggles between the menu and close icons and has aria-label/aria-expanded. Menu links close the panel when tapped. For signed-in users the menu shows "Open dashboard" instead of "Get started".
- Not added: a menu on the public business page (/#/p/slug). It has no navigation today; decide whether visitors should get one.

## Header menu and dashboard responsive (third pass)
- Header menu button: confirmed present at all widths. Previews previously showed it blank because the icon stub drew nothing; previews now use real icons (react-icons).
- Dashboard: appended a responsive block to src/index.css.
  - 1050px and below: narrower sidebar column.
  - 850px and below: stats in 2 columns, editor in a single column, header wraps.
  - 620px and below: stacked header and actions, 2-column stats, readable sidebar and stat text, full-width link rows and action buttons, 16px form inputs (stops iOS zoom-on-focus).
  - 380px and below: stats in one column.
  - Grid and flex children allowed to shrink (min-width: 0) to avoid horizontal overflow.
- Not verified by rendering at a phone width. Checked by reading the CSS and rendering markup only.

## New pages (fourth pass)
- New file src/pages.tsx with: templates, examples, analytics, billing, settings, login, signup, onboarding (3-step), help (FAQ), contact, pricing, terms, privacy, changelog, and a 404 page.
- App.tsx: those views are routed through ExtraPage; unknown #/ hash paths open the 404 page; the homepage footer links to templates, examples, pricing, help, contact, terms and privacy.
- index.css: .xp-* styles for the new pages.
- Not built: lead capture (listed under Pro but not implemented, so no page was added for it).
- Placeholder content, clearly labelled in the UI: analytics, billing and examples use sample data; terms and privacy are draft placeholders that need reviewed legal text; the contact form does not send anything; the changelog lists only the changes in this build.

## Menu arrangement (fifth pass)
- Desktop bar: Features, Templates, Pricing, Services, About, Help. Templates and Pricing now open their pages.
- Menu panel grouped into Product (Features, Templates, Examples, Pricing), Company (About, Services, Contact, Changelog) and Help (Help center, Terms, Privacy), with an Account section at the bottom.
- Account section: Open dashboard and Sign out when signed in; Log in and Get started when signed out.
- Desktop: the menu opens as a dropdown panel under the header. Below 1080px the bar links are hidden and reached through the menu.

## Collapsible menu groups (sixth pass)
- Product, Company and Help in the menu are now dropdowns. Each header shows the group name, the number of links and a chevron. Tapping it opens or closes the group.
- Only one group is open at a time. All groups start closed.
- The Account section stays visible at the bottom of the menu.
- Removed the link count from the menu group headers. The chevron is unchanged.

## Menu on the new pages (seventh pass)
- The menu panel data (MENU_GROUPS, MenuItem) moved from App.tsx to pages.tsx so the homepage and the new pages share one menu.
- The new pages' header has a menu button that opens the same collapsible groups and the Account section.
- Desktop: the panel is a dropdown under the header. Phone: it opens below the header.

## Extra page header (eighth pass)
- Removed the Templates / Examples / Pricing / Help row from the header of the new pages. Those links are in the menu and the footer.
- The header now shows the logo on the left, with Log in (or Dashboard) and the menu button on the right, on one row at all widths.

## Phone hero layout (ninth pass)
- Below 620px the homepage phone mockup is now part of the normal page flow, so the box grows to fit it. Before, it was centred in a fixed-height box and overflowed up over the avatar row and down toward the footer.
- The "Your page is live" and "Page insights" badges sit above and below the card, not over its text.
- Desktop layout is unchanged.

## Currency display for plan prices (tenth pass, display only)
- New src/currency.tsx: currency table (NGN, USD, GBP, EUR), country-to-currency detection from the browser language (then timezone, then USD), formatPrice(), and a CurrencyPicker.
- Plans now carry a naira amount (ngn). Prices are converted for display on the homepage pricing section, the signed-in plans view and the Pricing page.
- A picker above each plan grid lets visitors change the currency. Non-naira prices show a note: "Indicative price. Checkout currency: NGN."
- Billing invoices stay in NGN, since charges are in NGN.
- PLACEHOLDER RATES: the perNaira values in src/currency.tsx are not real rates. Replace them before launch.
- Not done: IP-based country detection (needs a server-side country header) and any checkout in other currencies (needs a payment provider).

## Dashboard tools (eleventh pass)
- Overview: "Page checklist" (7 checks: name, headline, bio, photo, visible link, service, contact) with a Fix button that jumps to the right tab, and "Share kit".
- Share kit: Share on WhatsApp (opens a message with the page link), Copy page link, and Download backup (the profile as a JSON file).
- Page builder: each link has Move up, Move down and Duplicate buttons. Move buttons are disabled at the ends of the list.
- Everything uses the existing save path (PUT /api/site). No backend changes.

## Preview fix: uploads not showing (twelfth pass)
- No source changes. The live preview's API stub returned only {ok: true} for uploads, so the logo and background were sent but never displayed. The stub now returns the path and an image or video URL, the same shape as the real backend, so uploads show in the editor and on the public page.
- Preview only: uploads are kept for the session. Reloading the page resets them, because the preview has no database.

## Templates in the dashboard (thirteenth pass)
- New src/templates.ts: six starter templates (Studio, Kitchen, Consultant, Shop, Creator, Clinic). Each sets an accent colour, a theme, a starter headline, bio, links and services.
- Page builder: a Templates panel above Page appearance. Choosing a template sets the colour and theme, and fills in the headline, bio, links and services only where they are empty.
- The Templates gallery on the site now reads from the same list.
- Styles added for the public-page themes "gradient", "minimal" and "bold". These had no CSS before, so choosing them in Page appearance looked the same as Midnight. They now look different.

## Sign-up page (fourteenth pass)
- Sign-up page replaced: name, business name, email, page link (validated; reserved words blocked), and a required terms checkbox. The button shows errors after a submit attempt.
- New src/signup.ts: validation rules (same slug rule as the backend), and storage of the sign-up details until sign-in returns.
- Setup after sign-in: if the account is brand-new (default profile, no links), the app saves the business name, page link and email with PUT /api/site. If the link is taken, the server's message is shown and the user can choose another in the page builder.
- Returning users are not affected: the saved details are discarded if the account already has a page.
- "Get started", "Create your free page", "Build your page", "Choose a plan", and the About and Services get-started buttons now open the sign-up page. "Log in" still signs in directly.
- The page-link availability is checked when the page is saved, not on the form. There is no availability endpoint in the backend.
- Not tested in a browser: the sign-in round trip, the setup step, and the terms links.

## Affiliate programme (fifteenth pass, frontend only)
- Affiliate page: how it works, a programme-terms box marked as a draft (commission rate, payout schedule, payout method and referral window still to be confirmed), and a referral section.
- Signed-in users see their referral link (page link with ?ref=), with Copy and Share on WhatsApp. Signed-out visitors see a Create my page button.
- Referral capture: a ?ref=<page-link> on the site is saved, carried through sign-up, and stored on the new page as referredBy. The referral is ignored if it points to the same page.
- Navigation: Affiliate is in the menu (Company group), the footer, and the dashboard sidebar.
- Not built: referral tracking, earnings, payouts and any affiliate numbers. The referral link is not verified on the server, and referredBy is a client-written field until the backend checks it. Commission terms are not set.

## Blank preview fix (sixteenth pass)
- Bug: the affiliate work added a hook (referralUrl, useMemo) after the App's loading-screen return. The app loaded, then crashed on its second render and showed a blank page. The hook now sits above that return. Nothing else changed.
- The preview now shows an error message on the page if the app fails to start, instead of a blank screen.

## Logo (seventeenth pass)
- The "L" tile and the "linkora." text are replaced with the uploaded logo on every page: headers, dashboard sidebar, loading screen, 404 page, the about and services art, and both footers.
- Headers and the sidebar show the star mark only. Footers and the Services art show the full logo with its tagline.
- src/brand.tsx: BrandMark picks the white version on dark pages and the black version on the light dashboard. The word "Linkora" stays as the accessible name.
- Assets in src/assets (transparent PNGs made from the uploaded file). src/assets.d.ts declares the image imports for TypeScript.
- Browser tab icon: public/favicon.png, linked from index.html.
- The mock profile card on the About page still shows an "L" avatar. It is sample content, not the site brand.

## Name and logo size (eighteenth pass)
- Site name changed to MOVVE in all visible text: page title, headers, footers ("Made with MOVVE."), copyright, owner console, About and Services copy, the logo's accessible name, and the backup file name.
- Internal identifiers keep their old names (localStorage keys such as linkora-signup-draft, CSS animation names, database table names), so existing data and styles are unaffected.
- "movve" is reserved as a page link, as is "linkora".
- Logo made larger. At 44px the halftone dots blurred into a solid shape and the footer tagline was too small to read. Header mark is now 64px (56px on phones), footer logo 260px (200px on phones).

## Name changed to mavvlink (nineteenth pass)
- Site name is now mavvlink. Running text uses "mavvlink"; all-caps labels (eyebrows, "YOUR … PAGE", the copyright line) use "MAVVLINK".
- Page title, logo alt text, footers, backup file name and reserved page links updated. The old name stays reserved.
- Internal identifiers unchanged (localStorage keys, CSS animation names, database table names).

## Header and footer wordmark (twentieth pass)
- The header and footers now show the name "Mavvlink" as a text wordmark instead of the star logo. This covers the homepage, the About and Services pages, the dashboard, and the extra pages (header and footer).
- The star logo stays on the loading screen, the public "Page not found" screen, and the art on the About and Services pages.
- src/brand.tsx: new Wordmark component. Styles in index.css under "Wordmark".

## Multi-column footer (twenty-first pass)
- New src/footer.tsx: SiteFooter. Brand block with the wordmark, a one-line description and sign-up / log-in buttons (dashboard button when signed in). Three link columns: Product (Features, Templates, Examples, Pricing, Affiliate), Company (About, Services, Contact, Changelog), Support (Help center, Terms, Privacy). Bottom bar with the copyright, Terms, Privacy and Sign out when signed in.
- Used on the homepage, the About page and the extra pages, replacing the three earlier footers.
- AboutPage now receives go, onLogin, signedIn and onSignOut props for the shared footer.
- The copyright line reads "© 2026 MAVVLINK. All rights reserved." Add the company's legal name here before launch if it differs.

## Footer trust row (twenty-second pass)
- A row at the very bottom of the footer: a green status dot linked to the changelog ("Platform status and updates") and a shield icon with "Your data is protected." linked to the privacy policy.
- The wording makes no claims about security certifications or live service status. Confirm any such claim before adding it.

## Footer email signup (twenty-third pass)
- Footer: "Get updates and tips by email" with an email box and a Subscribe button. Client-side validation, a saving state, and an error or thanks message. Shown on every page.
- Backend: POST /api/newsletter stores each address once in the mavvlink_subscribers collection and returns ok for repeats without revealing whether an address is already subscribed. Invalid addresses get a 400 with a message.
- Not built: sending emails, unsubscribe links, double opt-in, and rate limiting on the route. Connect a mail provider before sending anything to subscribers, and add an unsubscribe link to every email.
- Tested: the route logic with a mock database. Not tested against the real database, which is not reachable from this environment.

## Demo video section (twenty-fourth pass)
- New "See it in action" section on the homepage, between Features and Templates: the screen recording of a mavvlink page in a phone frame, muted and looping, with playback controls, and a "Create your free page" button.
- The upload (HEVC, 886x1920, 6.3 s, with audio) was converted to H.264 at 480x1040, audio removed, 229 KB. HEVC does not play in every browser; H.264 does.
- The clip shows the editor's "Back to editor" button on the page. Re-record it without that button before launch if it should not appear on the marketing site.
- Imported as src/assets/demo.mp4 (assets.d.ts declares *.mp4).

## Demo video playback fix (twenty-fifth pass)
- The video was embedded as a very large data: address, which some phone browsers refuse to play. src/demo.tsx now turns the embedded data into a Blob address, with the original address as a fallback and a short message if both fail. Hosted builds use the file address directly.
- The video file itself was verified: it decodes all frames without errors and its index is at the start of the file.

## Video re-encode (twenty-sixth pass)
- demo.mp4 re-encoded as H.264 Constrained Baseline, level 3.0, 30 fps, with a silent AAC track, for players that reject other profiles or missing audio. 226 KB.
- demo-video.mp4 is delivered separately so the clip can be opened on its own.

## Demo phone size (twenty-seventh pass)
- The "See it in action" phone frame is smaller: 240px wide at most (was 300px), and 62% of the screen width on phones (was 80%).

## Hero globe (twenty-eighth pass)
- The hero phone mockup and its orbit rings and floating badges are replaced by an animated globe (src/globe.tsx, canvas).
- The globe draws land as dots, faint latitude and longitude lines, glowing arcs between cities that pulse along their length, and a pulsing ring at each city. It rotates about 40 seconds per turn and stops for users with reduced motion.
- src/globe-data.ts holds hand-drawn, simplified continent outlines and 15 city positions. They are approximations for a stylised look, not survey-accurate geography. For accurate coastlines, replace LAND with Natural Earth 110m land data (public domain).
- The globe is drawn with plain canvas code, so it adds no new dependencies.

## Larger globe (twenty-ninth pass)
- Hero globe enlarged to 480px wide on large screens (was 360px), up to 92% of the screen width on phones. The hero area now grows to fit it instead of clipping it.

## Customer photos in the hero (thirtieth pass)
- The three letter placeholders (J, A, M) in the hero avatar stack are replaced with photos: src/assets/avatars/avatar-1.jpg, avatar-2.jpg and avatar-3.jpg (96px square). The "+" circle stays.
- Avatar circles increased from 26px to 34px so the faces read clearly.
- Before launch: confirm these people are real customers and have agreed to their photos being used. If they are not, the row implies endorsements that are not real.

## Footer buttons removed (thirty-first pass)
- The "Create your free page", "Log in" and "Open dashboard" buttons are removed from the footer's brand block. Sign-up and log-in are still in the header. The footer keeps the subscribe form, the link columns and the bottom bar.
- SiteFooter no longer reads the onStart and onLogin props; the callers can keep passing them.

## Vercel build fix (thirty-second pass)
- vite.config.ts: the build target was Chrome 87 / Safari 14, which the compiler cannot build for this code (array destructuring). Target is now es2020, Chrome/Edge 100, Firefox 110, Safari 15. Compiles.
- Still needed before a Vercel deploy: src/App.tsx and src/pages.tsx import '@appdeploy/client', which is not in package.json, so Vercel cannot resolve it. The backend (backend/index.ts) also uses the same platform and must be hosted somewhere else.

## Vercel build fix: platform SDK removed (thirty-third pass)
- The Vercel build failed with "Rolldown failed to resolve import" because src/App.tsx imported @appdeploy/client, a platform package that is not on npm.
- src/appdeploy-client.ts replaces it with a demo client. Sign-in, saving, uploads and the backend calls are simulated in the browser; nothing is stored on a server. The deployed site builds and runs as a demo.
- To go live, replace src/appdeploy-client.ts with a real auth and backend client, and deploy backend/index.ts (or its equivalent) somewhere that runs on Vercel.
- The browser build target fix from the previous pass is kept.

## Empty space under the globe (thirty-fourth pass)
- On phones, the hero's bottom padding and the globe area's bottom padding are removed when the globe is shown, so the globe sits directly above the divider.

## Logo in the Services art tile (thirty-fifth pass)
- The Services page's art tile is 200px (was 160px) and the full logo inside it is 160px wide (was 150px), so the star and tagline sit centred with space around them.
- The tagline is part of the logo artwork, so it stays small at this size.

## Centred logo in the Services tile (thirty-sixth pass)
- The full logo was off-centre because its star sits left and its tagline runs right. The tile now shows the star and tagline as two separate images (src/assets/tagline-white.png and tagline-dark.png), stacked and centred with BrandStack in src/brand.tsx.

## Services tile: star only, centred (thirty-seventh pass)
- The tagline is removed from the Services art tile. The tile shows the star alone, centred, at 96px wide. The tagline images and BrandStack are removed.

## Extra pages redesigned to match the homepage (thirty-eighth pass)
- Every extra page (Templates, Examples, Analytics, Billing, Settings, Log in, Sign up, Onboarding, Help, Contact, Pricing, Terms, Privacy, Changelog, Affiliate, 404) now opens with a homepage-style hero: eyebrow, large two-tone headline and an art panel.
- Illustrations are drawn in SVG and CSS, plus the star logo, the dotted globe and the demo video. Each page has its own art: shield (terms and privacy), lock (log in), envelope (contact), chat bubbles (help), steps (onboarding), card (billing), chart (analytics), globe (affiliate), faded star (404), and the demo video (examples).
- Template and example cards show mini page mock-ups. The pricing page highlights the middle plan. Changelog is a timeline with a pulsing dot.
- All forms and logic are unchanged: sign-up validation, onboarding steps, settings, currency picker, referral link and copy, and the footer subscribe form.
- Not changed: the dashboard and the public business page.

## Centred logo-strip label (thirty-ninth pass)
- "ONE LINK FOR EVERY PART OF YOUR BUSINESS" is centred on all screen sizes.

## Ask mavvlink assistant (fortieth pass)
- A vertical "Ask mavvlink" tab on the right edge of every page opens a chat panel. The tab breathes with a glow and its sparkle turns; replies type out word by word with a bouncing typing indicator.
- src/agent.tsx answers from a built-in knowledge base about this site: sign-up, pricing and plans (quoted from src/plans.ts), currency, custom domains, templates, links, sharing and backup, affiliate, analytics, newsletter, log in, terms, privacy, contact, help and the menu. Where useful, it offers a button that opens the right page.
- Questions outside the site get a polite fallback. The assistant does not promise features that are not built (lead capture is described as not yet available).
- Not built: a general AI model. Open-ended answers need an AI API key kept on the server (a backend route), which should be added separately before the assistant answers arbitrary questions.
- src/plans.ts now holds the plan list, used by both the pricing pages and the assistant. App.tsx's default export wraps AppInner so the assistant renders on every page.

## Slimmer assistant (forty-first pass)
- The Ask tab is narrower and shorter (10px text, smaller padding and corner radius), and the chat panel is 312px wide by 440px tall at most. Icons and inputs are scaled to match.

## Slimmer assistant (forty-first pass)
- The Ask tab is narrower and shorter (10px text, smaller padding and corner radius). The chat panel is 312px wide by 440px tall at most. Icons, text and inputs are scaled to match.

## Centred footer bottom bar (forty-second pass)
- The copyright line and the Terms and Privacy links in the footer's bottom bar are centred.

## Phone mock-up replaces the demo video (forty-third pass)
- The demo video is replaced with PhoneMockup (src/demo.tsx): an HTML and CSS phone showing an example mavvlink page (Studio North, three links, a featured service, and the footer line). Its content matches the site's templates, and it animates in with a sheen across the service card.
- The phone is smaller: 196px wide at most (was 240px), 54% of the screen on phones (was 62%).
- The Examples page uses the same phone mock-up in place of the video.
- DemoVideo is no longer used but remains in src/demo.tsx.

## Animated buttons in the phone mock-up (forty-fourth pass)
- The three link buttons in the phone mock-up keep moving: a light sweep crosses each one, its green glow pulses, and its arrow nudges. The buttons start at staggered times. Motion is turned off for users who have reduced motion enabled.

## Eye avatar on the About page (forty-fifth pass)
- The green "L" avatar in the About page's profile card is replaced with the eye image (src/assets/eye.gif). It is cropped to a square, shrunk to 96px and keeps its animation (108 KB). The avatar is now a circle with a thin green border.
- src/assets.d.ts declares *.gif imports.

## Dashboard sidebar no longer sticky (forty-sixth pass)
- The dashboard sidebar scrolls with the page on desktop and phones. Only the site header stays sticky.
- The page builder's phone preview (.preview-sticky) is unchanged.

## Dashboard sample name is Mavvlink (forty-seventh pass)
- The dashboard's sample page is now "Mavvlink" (slug mavvlink, email hello@mavvlink.example, owner owner@mavvlink.example and "Mavvlink Owner"). The Settings page shows the same name, email and public link (/p/mavvlink).
- Sample data is in src/appdeploy-client.ts, which is the demo client. Replace it with real data when the backend is connected.
- The sign-up and onboarding forms still use "Northstar Studio" as example placeholder text. Change those in src/pages.tsx if you want them updated too.

## Aurora template and full-screen preview (forty-eighth pass)
- New "Aurora" template (src/templates.ts): a blue-to-pink gradient page with a large uppercase name, a rounded profile tile, glass link buttons with the first link highlighted in gold, and a gold-bordered headline pill. It appears in the dashboard template picker and on the Templates page.
- Theme "aurora" in the public page (index.css): slowly shifting gradient background, floating avatar, staggered button entrance (motion off for reduced-motion users).
- The page builder's preview has a Full screen button. It opens the real public page filling the screen, with a Back button to return to the editor.
- The Aurora design is original. It does not reuse a third-party photo, name or logo. The social-icon row is not included, since profiles have no social-link field yet.

## Public page: centred, no light/dark switch (forty-ninth pass)
- The sun button that switched the public page between light and dark is removed. Each page now follows its chosen theme (including the light Clinic theme).
- The public page content is centred, and the Aurora name is smaller (13vw, up to 72px) so MAVVLINK fits on narrow screens without overflowing.

## Touch feedback on public page buttons (fiftieth pass)
- Link buttons on every public page theme shrink slightly when pressed, glow briefly, and show a light ripple from the centre. Motion is off for reduced-motion users.
- Aurora buttons keep their entrance animation with backwards fill so the press effect is not blocked.

## Products grid (fifty-first pass)
- Business pages can list up to 12 products: photo, name, link where customers buy, and an optional price. The Products section sits above Services in the Services view of the dashboard, with move up/down and remove controls.
- The public page shows the products in a two-column grid under the links. Each product is a link, and clicks are counted under "Product: <name>".
- Backend (backend/index.ts): products are sanitised (12 maximum, title 80 characters, link 500, price 40), any client-supplied image address is dropped, and product photos are stored and resolved the same way as the logo.
- Product photos are uploaded through the existing /api/upload route, which returns a path for the file. Check in the storage settings that these files are kept, since the route was built for the logo.
- Assistant: answers questions about adding products.
- Not tested against the real backend or storage. Test an upload, save, reload, and a product click before launch.

## Subscriber-only templates and products (fifty-second pass)
- src/access.ts holds the rules: a subscriber is a paid plan that is active and not expired. Subscribers get all templates and up to 12 products. Everyone else gets two templates (Studio and Consultant, set in FREE_TEMPLATE_IDS) and two products.
- Dashboard template picker: locked templates are marked "Subscribers" and cannot be applied; choosing one shows a message.
- Templates page: locked templates show a Subscribers badge and a "Subscribe to use" button that opens Pricing.
- Products: the Add product button is disabled at the limit, with a note on the free limit.
- Server (backend/index.ts): already limits saved products to 2 for non-subscribers and 12 for subscribers, using the server-held plan and subscription status.
- Template limits are enforced in the app. The server stores only a page's theme, not which template it came from, so a free account could still set a locked theme by calling the API directly. To close that, add a theme check to the save handler.

## Custom domains for Growth and above (fifty-third pass)
- Custom domains are available on the Growth plan and above (Pro includes everything in Growth). A subscription must be active and not expired. Free, cancelled and expired accounts cannot request one.
- Server (backend/index.ts): POST /api/domain returns 403 unless the saved plan is Growth or Pro, with an active, unexpired subscription.
- App (src/access.ts, src/App.tsx): the Custom domain page shows a locked panel with a "See plans" button to users who are not eligible. The request form is hidden for them.
- Plans (src/plans.ts): "Custom domain connection" moves from Pro to Growth.
- Help page, assistant and FAQ copy updated to match.
- To allow Growth only, change the rule in canUseCustomDomain (src/access.ts) and in the server check.

## Messages clear after 10 seconds (fifty-fourth pass)
- Notices (the green messages, including "Free selected. Save to confirm your plan preference.") clear themselves 10 seconds after they appear. The X button still closes them sooner.
- A repeated message does not restart the 10-second timer; it clears at the first 10 seconds after it first appeared.

## Auto-save (fifty-fifth pass)
- The dashboard saves the page automatically 1.5 seconds after the last change, using the same save request as the Save button. Nothing is saved on load.
- The app keeps the last saved copy as a save point, so only real changes are sent.
- If the page is edited while a save is in progress, the reply does not overwrite the newer edits; the new changes are saved next.
- A status label beside "Publish changes" shows "Saving…", "All changes saved", or "Not saved yet. Check your page link." (for example, when the page link is not yet valid). Auto-save errors do not pop up as messages.
- The Save and Publish buttons still work.

## Social icons on some templates (fifty-sixth pass)
- New Social profiles section in the Services view: links for Instagram, TikTok, X, YouTube, Facebook, LinkedIn and WhatsApp, plus a Show icons switch.
- Public page: a row of round icons under the links, shown only when Show icons is on and at least one profile is filled in. Each icon links to the profile in a new tab.
- Studio, Creator and Aurora templates turn Show icons on when applied; other templates leave it off.
- src/social.tsx: the platform list and drawn SVG icons (no icon-library dependency).
- Backend (backend/index.ts): socials are limited to the known platforms, links without a scheme get https://, and only http(s) links are kept. showSocials is stored as a boolean.
- Assistant answers questions about social icons.

## Sliding social logos on the homepage (fifty-seventh pass)
- A band of the seven social logos (Instagram, TikTok, X, YouTube, Facebook, LinkedIn, WhatsApp) slides continuously across the strip under "ONE LINK FOR EVERY PART OF YOUR BUSINESS". Each logo has a soft glow pulse. The slide pauses while the band is touched or hovered.
- The band fades at its edges. The list is repeated once so the loop joins without a gap. Repeats are hidden from screen readers.
- With reduced motion turned on, the logos sit still in a wrapped row.

## Social logo band on the About page (fifty-eighth pass)
- The sliding social logo band is also on the About page, between the top section and "Our purpose".

## Social band fixed and Telegram added (fifty-ninth pass)
- The homepage band now sits outside the logo-strip section, the same as on the About page. The strip's own styles were overlapping the logos and labels.
- Telegram added to the platform list, the social editor, the icon set, and the server's accepted platforms.
- YouTube was already in the list; it was hard to read because of the overlap, which is now fixed.

## Phone mock-up: new image and name (sixtieth pass)
- The star logo in the phone mock-up is replaced by a round animated image (src/assets/manuel.gif, cropped square, 96px, 84 KB).
- The mock-up's name is changed from "Studio North" to "Manuel". The Examples page uses the same mock-up, so it changes too.
- Confirm you have the rights to use the image before launch.

## Smoother motion across the site (sixty-first pass)
- Each page fades and slides in when it opens (dashboard, public pages and the other pages).
- Sections fade up as they scroll into view. The observer runs again on every page change, so newly opened pages animate too. Content stays visible if scripts do not run.
- Buttons have a light press effect and smoother colour and shadow changes.
- In-page links scroll smoothly.
- All of this is off for users with reduced motion turned on.

## Terms and conditions written (sixty-second pass)
- The Terms page now has 15 numbered sections: about these terms, accounts, your content, acceptable use, plans and currency, custom domains, the affiliate programme, newsletter, our service, intellectual property, suspension, liability, changes, governing law and contact. Each is a card with a contents bar at the top.
- The page carries a note that it is a draft and needs review by a qualified lawyer before launch.
- Governing law is set to the Federal Republic of Nigeria, based on the naira pricing. Confirm this, and the company details, before launch.
- The Privacy policy page still has placeholder sections.
