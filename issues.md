# AEU Documentation — Edge Cases, Gaps & Clarification Needed

This document identifies issues, ambiguities, and missing details that could cause a development AI to make wrong assumptions, build the wrong thing, or get stuck.

---

## 🔴 CRITICAL — Will Block Development

### 1. No Database/Backend Technology Stack Defined
The CRM doc defines *data models* and *field mappings*, but never specifies the actual backend stack. A dev AI needs to know:
- **Database**: PostgreSQL? MongoDB? Supabase? Firebase? PlanetScale?
- **Backend framework**: Node/Express? Next.js API routes? Django? Go?
- **Auth provider**: Clerk? Auth0? Supabase Auth? Firebase Auth? Custom JWT?
- **Payment processor**: Stripe? PayPal? Both? What handles escrow?
- **Push notification service**: Firebase Cloud Messaging (FCM) for Android is clear, but what about the APNs integration pattern? OneSignal? Expo Push?
- **Real-time layer**: WebSocket server? Supabase Realtime? Pusher? Socket.io? (Game Ticker, Live Ops Queue, and Surge all need real-time)
- **File storage / CDN**: Where do uploaded screenshots (OCR), sponsor assets, org logos go? S3? Cloudflare R2?
- **Email provider**: SendGrid? Resend? Postmark? (Transactional emails referenced everywhere)
- **OCR AI provider**: What service handles the OCR pipeline? Google Vision? AWS Textract? Custom model? OpenAI Vision?

**Why this matters**: Without this, a dev AI will pick whatever it wants and you'll end up with a Frankenstein stack.

### 2. No API Specification
The doc maps CRM fields to frontend hooks beautifully, but there's no actual API contract:
- What are the API endpoints? REST? GraphQL? tRPC?
- What's the request/response shape for each hook (useMe, useDashboard, useCtas, etc.)?
- Are these one-call-per-dashboard or do hooks make multiple parallel calls?
- How does pagination work for tournaments, notifications, transactions?
- Rate limiting strategy?

### 3. Mobile App: React Native vs. Other?
The monorepo mentions `/apps/mobile` and references React Native once, but the rest of the doc references the same React component library (`shadcn/ui`, `react-router`, `zustand`, `Tailwind`). Need to clarify:
- Is this **React Native** (Expo? bare?)
- Is this **React Native for Web** (shared codebase)?
- Or a **PWA/Capacitor/Ionic** wrapper around the React web app?
- `shadcn/ui` does NOT work in React Native — it's web-only. This is a direct conflict.
- `react-router` doesn't work natively in React Native — you'd need `react-navigation`.
- `Tailwind` needs `nativewind` or similar to work in React Native.

**This single ambiguity could derail the entire mobile architecture.**

### 4. Payment & Escrow Logic Not Specified
The doc mentions wallets, entry fees, rake, escrow, prize payouts, and withdrawals — but the actual payment flow is undefined:
- How does escrow work? Does AEU hold funds in a Stripe connected account? Custodial wallet?
- What is the rake percentage? It's referenced everywhere but never defined.
- How are prize payouts actually executed? ACH? Stripe payouts? PayPal?
- When does AEU actually charge the rake — at entry or at settlement?
- What happens when a tournament is canceled mid-play? Full refund? Partial? Token?
- Multi-currency support? The doc says USD everywhere but mentions international players.

---

## 🟡 HIGH — Will Cause Wrong Implementation

### 5. Conflicting Membership Tiers Across Documents
Part 1 (Master Spec) and Part 6 (Feature Matrix) have discrepancies:
- **Player Membership**: Part 6 lists Free + Gold (2 tiers). Part 1 references "Gold" only. But Part 6 has no "Platinum" for players — is there one?
- **TO Membership**: Part 6 lists Free + Gold + Platinum with specific prices. Part 1 also references these. But the pricing in Part 6 ($29.99/$97.99/mo) doesn't appear in Part 1. Which is canonical?
- **Org Membership**: Part 6 says Gold + Platinum (no Free tier). But the Org Workflow in Part 4 talks about org discovery as if there's a free entry point. Can orgs exist without paying?
- **Sponsor Membership**: Part 1 says "Base (Free) + Platinum (Enterprise)". Part 6 has no Sponsor section in the feature matrix. What are the actual sponsor pricing tiers?

### 6. Token Economics Are Undefined
The doc repeatedly says tokens are critical to the entire platform, but:
- **Token-to-USD conversion rate**: Never defined. How many tokens = $1? This affects every hybrid entry calculation.
- **Earn rates per action**: "Tokens for wins, streaks, MVP" — but how many? 5 tokens? 500?
- **Token price (purchase)**: Can players buy tokens? At what rate?
- **Breakage model**: What % of earned tokens are never redeemed? (This is critical for financial modeling)
- **Inflation control**: What stops the economy from inflating if too many tokens are earned?
- The doc explicitly calls this out as "The One Risk to Stay Ahead Of" but doesn't provide the actual numbers needed to build it.

### 7. Dispute Resolution Process — Only Triggering Defined, Not Resolution
The CTA trigger map defines *when* a dispute fires, but:
- Who resolves disputes — the TO? AEU admin? Both?
- What evidence is submitted? Screenshots only? Video? Text?
- What's the resolution timeline? The doc says `cta_expiry_at = evidence window` — but what is that window?
- Can disputes be escalated? The doc mentions "escalated to admin" but doesn't define escalation rules.
- What happens to entry fees during a dispute? Held? Released to winner?
- Is there an appeal process?

### 8. "Automated Tournaments" Referenced but Never Defined
Part 6 mentions "(Automated tournaments) (tokens)" next to free tournament entries for Gold players. This implies AEU runs its own automated/system-generated tournaments, but:
- What are automated tournaments? Bot-matched? System-scheduled?
- How are brackets auto-generated?
- Who is the "TO" for an automated tournament — the system?
- How do prizes work for automated events?

### 9. Streaming Feature — Massively Under-Specified
Step 10 of the Player Workflow and multiple sections reference streaming:
- "Mobile App native streaming" — built-in streaming? To where? AEU's own servers?
- "Twitch/YouTube/Kick embeds" — embedded *where*? In the match room?
- "Stream overlay clickable" — who produces the overlay? How is it injected?
- "Premium viewing paywalled" — premium viewing of *what*? AEU-hosted streams?
- "Ad breaks / commercial breaks" — who serves these ads? What ad network?
- This is essentially an entire separate product (a streaming platform) mentioned casually.

### 10. No Error Handling for Financial Failures
The doc defines what happens when payments fail (P0 CTA), but:
- What about double-charges? (Player charged twice for one entry)
- What about partial escrow funding? (TO funded 80% of prize pool)
- What about withdrawal failures after payout? (Bank rejects ACH)
- What about chargeback disputes from Stripe/PayPal?
- What about refund policies? (No cancellation/refund policy is defined anywhere)

---

## 🟠 MEDIUM — Could Cause Confusion or Inconsistency

### 11. Season System Never Defined
Multiple references to "90-day seasons," "season XP," "season tier," "season reset" — but:
- When do seasons start and end? Calendar-based? Rolling?
- What resets vs. what persists between seasons?
- Are there season rewards? What are they?
- Is there a season pass concept?

### 12. Matchmaking System Not Specified
"Ranked matchmaking access" is a Gold feature, but:
- What matchmaking algorithm is used? Elo? Glicko? TrueSkill?
- How are matches paired? Same rank only? Skill-based?
- Is matchmaking separate from tournament brackets?
- What's the queue system? How long do players wait?

### 13. Leaderboard Calculations Undefined
Multiple leaderboards referenced (player, token, org, category, TO) but:
- What metric drives each leaderboard? XP? Win rate? Earnings?
- How often are they updated? Real-time? Daily? Weekly?
- What's the tiebreaker logic?
- Are there separate leaderboards per game?

### 14. KYC/W-9 Provider Not Specified
KYC is referenced as a gate for payouts:
- What KYC provider? Persona? Jumio? Plaid Identity? Stripe Identity?
- What verification level? Name+DOB? ID scan? Selfie?
- Is KYC required for all users or only at payout threshold?
- What's the flow on mobile for document upload?

### 15. Bracket Formats Only Partially Listed
"Standard bracket formats" (single/double elim) mentioned for Free TOs, "advanced formats" locked for Gold+:
- What are the "advanced formats"? Swiss? Round robin? Groups into bracket?
- How are seedings determined?
- How are byes handled?
- What about team vs. solo bracket differences?

### 16. Notification Frequency / Throttling Not Defined
The CTA trigger map has 80+ notification types. Without throttling:
- A player in 3 active tournaments could get 15+ push notifications in one day.
- There's no defined daily push cap, quiet hours, or batching strategy.
- What's the priority when a user has 5 active P0 CTAs simultaneously?

### 17. Multi-Game Support Assumptions
The doc assumes players can enter tournaments for multiple games, but:
- Are gamertags validated per platform? (PSN, Xbox, Steam have different ID formats)
- Can a player use different gamertags for different games?
- How does cross-platform matching work for games with cross-play?

### 18. Data Migration / Existing User Import
No mention of:
- How beta users are onboarded
- Whether there's an import from existing tournament platforms (Battlefy, Challonge, Start.gg)
- How historical data feeds into profiles

---

## 🔵 LOW — Worth Clarifying Before Scale

### 19. Affiliate Attribution Window
- How long does the referral cookie/attribution last? 24h? 30 days? Forever?
- If a player signs up free via a referral link but converts to Gold 3 months later, does the referrer still get credit?
- Can attribution be reassigned if a second referrer's link is clicked?

### 20. Accessibility on Mobile
- VoiceOver/TalkBack support required?
- The Web spec mentions WCAG AA — does this apply to the native app too?
- Haptic feedback patterns are referenced but not defined.

### 21. Content Moderation
- No moderation system defined for: team names, org names, gamertags, org bios, sponsor copy.
- What about toxic behavior in match rooms?
- No report/flag system mentioned.

### 22. Localization Scope
- "i18n readiness" mentioned but what languages at launch? English only?
- RTL mentioned as "eventually" — is this launch scope?
- Currency localization — is everything USD at launch?

### 23. Free-to-Gold "Monthly Free Entry Cap" Never Quantified
- Gold players get "free tournament entries (monthly cap)" — but how many? 1? 3? 5?
- Does this reset on billing date or calendar month?
- Does it carry over?

### 24. Org Prize Split Enforcement
- "Players must pre-agree to prize split parameters" — but what if they don't?
- Can the split be changed mid-tournament?
- What if a player leaves the org after winning but before payout?

### 25. Web3 Bridge Architecture
- The token strategy doc says "design the conversion bridge NOW" but provides zero technical spec.
- The token ledger needs to be blockchain-compatible from day one — but no ledger schema is provided.
- This could force a complete token system rewrite later if not addressed.

---

## Summary: Top 5 Things to Decide Before Giving This to a Dev AI

1. **Pick and specify the full backend tech stack** (database, auth, payments, hosting, push, real-time, file storage, email)
2. **Define the API contract** (endpoints, request/response shapes, auth headers, error formats)
3. **Resolve the mobile app framework** (React Native vs. Capacitor vs. PWA — this changes everything)
4. **Set token economics numbers** (earn rates, spend rates, USD conversion, purchase price)
5. **Define the payment/escrow flow** (Stripe Connect? Custodial? Rake timing? Refund policy?)

Without these five, a dev AI will be forced to make architectural assumptions that will be expensive to undo later.
