# AMATEUR ESPORTS UNION — Complete Documentation

---


---

# PART 1: UI/UX + Frontend Master Specification (v3)

**AMATEUR ESPORTS UNION**

UI/UX + Frontend Master Specification + Execution Tracker

*Version 3.0 \| Website + Telegram Mini App \| February 2026*

*Audience: Design \| Frontend Engineering \| Product \| Revenue \| Executive \| Confidential*

  -------------------------------------------------------------------------------------------------------------------
  **Category**        **Detail**
  ------------------- -----------------------------------------------------------------------------------------------
  Covers              UI/UX Spec, React Component Spec, Monetization Logic, Full Execution Tracker

  Surfaces            Web App + Telegram Mini App (shared design system)

  Architecture        React + TypeScript, Zustand, React Query, Tailwind, shadcn/ui

  Source Docs         AEU_CRM_Architecture_v2, AEU_UISpec_v1, AEU_ReactSpec_v1, AEU_Member_Workflows + Gamification

  Tracker Status      ☐ = Not Started ☑ = Complete --- = In Progress

  Classification      Confidential --- Internal Use Only
  -------------------------------------------------------------------------------------------------------------------

# **Section 1 --- Architecture Foundations**

## **1.1 The Core Rule: Every Screen Must Resolve Three Questions**

Before any UI renders --- web or Telegram --- the system must determine three things. Failure to enforce this logic produces confusing dashboards, lost entry fees, missed renewals, low retention, and broken monetization loops.

  -------------------------------------------------------------------------------------------------------------
  **Question**                      **UI Resolution**                   **Revenue Resolution**
  --------------------------------- ----------------------------------- ---------------------------------------
  Who is viewing?                   Role-aware dashboard layout         Monetization tier + entitlement check

  Where are they in the workflow?   Context-specific layout + panels    Revenue trigger alignment

  What is the next best action?     Single primary CTA above the fold   Protects or drives revenue
  -------------------------------------------------------------------------------------------------------------

> *These three questions mirror the CRM event logic from AEU_CRM_Architecture_v2, expressed as UI composition and priority ordering rules.*

## **1.2 Priority Tiers --- Visual Urgency + Revenue Alignment**

Every UI element, card, and notification must be assigned one of four priority tiers. These tiers govern layout dominance, styling intensity, placement, and interruption level.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Tier**   **Label**                  **UI Behavior**                                               **Monetization Impact**              **Example**
  ---------- -------------------------- ------------------------------------------------------------- ------------------------------------ ------------------------------------------
  P0         Critical / Time-bound      Full takeover card; includes countdown; cannot be buried      Directly protects existing revenue   Check-in, payout failure, payout blocked

  P1         Action Soon / Contextual   Visible on home feed + relevant screen; not a full takeover   Drives engagement revenue            Tournament reminder, roster warning

  P2         Transactional / Records    Visible in Activity/Receipts/Billing only                     Protects subscription MRR            Membership renewal notice

  P3         Engagement / Nudges        Non-blocking cards, banners, badges                           Drives retention + token usage       Streak badge, affiliate nudge
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *If a P0 event is not visually dominant above the fold, it is a revenue risk. P0 items must never be buried in feeds or tabs.*

## **1.3 Two Surfaces, One Design System**

  ------------------------------------------------------------------------------------------------------------------
  **Capability**           **Web App**                                     **Telegram Mini App**
  ------------------------ ----------------------------------------------- -----------------------------------------
  Navigation               Left sidebar (desktop) / bottom tabs (mobile)   Bottom tabs only --- max 2 screens deep

  Tables / dense data      Full tables with sorting, pagination            Card stacks + collapsible sections only

  Multi-panel dashboards   Yes --- full multi-column layouts               Compressed single-column stacked cards

  Analytics / charts       Full charts                                     KPI tiles only --- no chart libraries

  Confirmations            Modal overlay                                   BottomSheet

  Heavy animations         Limited and purposeful                          Minimal
  ------------------------------------------------------------------------------------------------------------------

> *Shared across both surfaces: typography scale, icon set, color tokens, spacing system, component library, interaction patterns, and all UI states (loading, empty, error, offline, permission denied, feature locked).*

# **Section 2 --- Monorepo, Tech Stack & Route Map**

## **2.1 Monorepo Layout**

> /apps
>
> /web --- AEU website (React)
>
> /telegram --- Telegram Mini App (React, same route map)
>
> /packages
>
> /ui --- shared components
>
> /tokens --- design tokens + Tailwind preset
>
> /core --- types, role logic, CTA engine, utilities, platform adapter
>
> /api --- client SDK layer + React Query hooks

## **2.2 Tech Stack**

  ----------------------------------------------------------------------------------------------------------
  **Concern**     **Library**                                    **Notes**
  --------------- ---------------------------------------------- -------------------------------------------
  Framework       React + TypeScript                             All components fully typed --- no \'any\'

  Routing         react-router (web) + thin wrapper (Telegram)   Same route map on both surfaces

  App state       zustand                                        Role context, CTA engine, UI state

  Server state    \@tanstack/react-query                         All API calls, caching, mutations

  Styling         Tailwind + shared token layer                  Tokens are single source of truth

  Forms           react-hook-form                                All multi-step forms

  Validation      zod                                            Forms + API response validation

  UI library      shadcn/ui (recommended)                        Speed + consistency; swap freely
  ----------------------------------------------------------------------------------------------------------

## **2.3 Full Route Map**

  --------------------------------------------------------------------------------------------------------------------------------------------------------
  **Route**                                  **Roles**                                          **Notes**
  ------------------------------------------ -------------------------------------------------- ----------------------------------------------------------
  /welcome + /auth + /onboarding             Public                                             Auth + 3--5 step profile stepper; save-as-you-go

  /home                                      All roles                                          Role-aware dashboard --- routes internally by activeRole

  /tournaments + /tournaments/:id            All                                                Browse, filter, quick register, detail, entry CTA

  /match/:matchId                            Player, Captain                                    Check-in, lobby, result submit, confirm/dispute

  /teams + /teams/:teamId                    Player, Captain                                    Browse, create, roster, record

  /org                                       org_admin                                          Org ops --- team health, caps, budget, bulk reg

  /wallet                                    All                                                Token + fiat wallet, transactions, withdraw, top-up

  /marketplace                               All (phase gated)                                  Token redemption, gear, merch, sponsor drops

  /notifications                             All                                                Action Required / Updates / Receipts tabs

  /profile + /settings                       All                                                Identity, preferences, membership management

  /to/ops + /to/events/:id                   to                                                 Live Ops Queue + event pipeline + creation

  /sponsor/campaigns/:id + /sponsor/assets   sponsor                                            Campaign KPIs + creative asset management

  /billing                                   Tournament Organizer, sponsor, org_admin, player   Billing, invoices

  /admin                                     aeu_admin                                          Internal --- cross-role monitor, incident tools
  --------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 3 --- Role Architecture & CTA Engine**

## **3.1 Role Types, Stacking & Switcher**

A user can hold multiple roles simultaneously. The UI renders one active role context at a time, switchable via the Role Switcher in the global header.

  ---------------------------------------------------------------------------------------------------------------------
  **Role**         **Default Home View**                                **Notes**
  ---------------- ---------------------------------------------------- -----------------------------------------------
  player           Player dashboard --- tournaments, wallet, team       Default for all users

  team_captain     Player dashboard + roster ops panel                  Overlay on player --- same home, extra panels

  org_admin        Org dashboard --- team health, seat caps, bulk reg   Separate view

  to               Ops Console --- live queue, events pipeline          Separate view

  sponsor          Sponsor dashboard --- campaigns, assets, KPIs        Separate view

  aeu_admin        Admin console --- system-wide management             Internal only
  ---------------------------------------------------------------------------------------------------------------------

> // zustand role store contract
>
> activeRole: Role // currently rendered role
>
> roles: Role\[\] // all roles this user holds
>
> setActiveRole(role: Role) // switches the UI context --- never changes identity/profile

### **Role System --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                                                   **Implementation Notes**
  --------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------
  ☐ Multi-role stacking (player + captain + org + TO + sponsor)   Single user account holds multiple roles; all available at login

  ☐ Role switcher component                                       Dropdown in TopBar; shows all roles the user holds; fires setActiveRole()

  ☐ Role-scoped navigation trees                                  NavItem.roles\[\] filters nav items per activeRole on every render

  ☐ Role-scoped dashboard layouts                                 HomePage routes internally: \<PlayerHome\>, \<CaptainHome\>, \<OrgAdminHome\>, \<TOHome\>, \<SponsorHome\>

  ☐ Role-scoped CTA filtering                                     CTA Engine filters by activeRole before priority sort

  ☐ Role-aware notification routing                               Notifications tagged by role; center filters by activeRole

  ☐ Workflow stage tagging on every screen                        Every screen has a WorkflowStage tag used by CTA Engine for context filtering
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.2 CTA Engine --- Frontend Composition Layer**

The CTA Engine is the most critical piece of frontend architecture. It decides what appears in the Next Best Action card, the Notifications center, and all inline contextual CTAs. Without it, revenue protection and user guidance break down.

### **CTA Object Contract**

> type CTA = {
>
> id: string;
>
> priority: \'P0\' \| \'P1\' \| \'P2\' \| \'P3\';
>
> stage: WorkflowStage;
>
> role: Role;
>
> title: string;
>
> body?: string;
>
> primaryAction: { label: string; to: string; };
>
> secondaryAction?: { label: string; to?: string; kind?: \'dismiss\'\|\'snooze\'\|\'learn_more\'; };
>
> expiresAt?: string; // ISO 8601 --- drives Countdown
>
> context?: { type: \'tournament\'\|\'team\'\|\'membership\'\|\'wallet\'\|\'marketplace\'\|\'system\'; id?: string; };
>
> state: \'open\' \| \'completed\' \| \'dismissed\' \| \'expired\';
>
> channelsSent?: string\[\]; // cta_channel_sent\[\] --- suppresses re-send
>
> };

### **CTA Selection Rules**

-   Filter by activeRole

-   Filter where state === \'open\'

-   Filter by workflow stage of current screen

-   Sort by priority (P0 first), then expiresAt soonest

-   Top result → \<NextBestActionCard /\>

-   Remainder (excluding top) → \<CTAFeed /\>

### **CTA Engine --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                                                      **Implementation Notes**
  ------------------------------------------------------------------ ---------------------------------------------------------------------------------------------------
  ☐ Central CTA Provider                                             Context provider wrapping the app; runs selection logic on every role/state change

  ☐ P0 takeover layout (Web + Telegram)                              Full-width high-contrast card; countdown; no dismiss; overrides all other top-slot content

  ☐ P1 contextual card                                               Standard card in CTAFeed; dismissible; visible on home + relevant screen

  ☐ P2 transactional banner                                          Slim banner; visible in Activity/Receipts/Billing area only

  ☐ P3 engagement card                                               Non-blocking; appears in feed below P0/P1; snooze allowed

  ☐ Countdown support (cta_expiry_at)                                \<Countdown /\> renders inside NBA card when expiresAt is set; ticks live; fires onExpire refetch

  ☐ CTA state transitions (open → completed → expired → dismissed)   State machine in zustand; mutations update server + local state

  ☐ Channel suppression using cta_channel_sent\[\]                   CTA not re-shown in channel (push/in-app/email) if that channel is in channelsSent\[\]

  ☐ CTA workflow stage filter                                        CTAs filtered by current screen\'s WorkflowStage tag

  ☐ Deep link routing from Telegram push                             Platform adapter opens correct Mini App route from push notification deep link
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.3 Feature Gating --- GateResult Contract**

> type GateResult =
>
> \| { allowed: true }
>
> \| { allowed: false; reason: \'membership\'\|\'role\'\|\'kyc\'\|\'w9\'\|\'seat_cap\'\|\'team_cap\'\|\'platform\'; required?: string };

  ---------------------------------------------------------------------------------------------------------------------------
  **Gate Reason**   **Trigger**                             **User Sees**
  ----------------- --------------------------------------- -----------------------------------------------------------------
  membership        Action requires higher tier             LockedFeatureCard: feature name, 3 benefits, price, Upgrade CTA

  role              Current role cannot do this             PermissionDeniedState + role switch prompt

  kyc               KYC not completed                       Inline KYC prompt with steps

  w9                W-9 not submitted (\>\$600 threshold)   P0 CTA + block withdraw only

  seat_cap          Org seat limit reached                  OrgCapMeter at 100% + upgrade prompt

  team_cap          Org team limit reached                  Same as seat_cap

  platform          Feature unavailable on this surface     Short inline message + link to web
  ---------------------------------------------------------------------------------------------------------------------------

# **Section 4 --- Global Design System & Website-Specific Components**

## **4.1 Design Tokens (Shared Across All Surfaces)**

Tokens are the single source of truth for all visual decisions. No hardcoded colors, font sizes, or spacing values should appear in component code. All tokens live in /packages/tokens.

  --------------------------------------------------------------------------------------------------------------
  **Token Category**     **Scale / Values**                                    **Usage**
  ---------------------- ----------------------------------------------------- ---------------------------------
  Color --- background   background, surface, border                           Page bg, card bg, dividers

  Color --- text         text, muted, inverse                                  Body text, labels, text on dark

  Color --- status       success, warn, error, info                            Badges, alerts, inline states

  Color --- accent       accent-primary, accent-secondary                      CTAs, highlights, active nav

  Type scale             Display / H1 / H2 / H3 / Body / Caption / Label       All text across all components

  Spacing                4-pt base: 4, 8, 12, 16, 24, 32, 48, 64               Padding, margin, gap everywhere

  Border radius          12--16px cards; 8px inputs; 999px pills               Consistent rounding

  Shadow                 soft (card), elevated (modal/dropdown)                Depth and layering

  Motion                 fast 150ms, standard 250ms, slow 400ms --- ease-out   All transitions
  --------------------------------------------------------------------------------------------------------------

> *Token names must be semantic (e.g. \'color-surface\' not \'gray-100\'). This allows theming without touching component code.*

## **4.2 AppShell --- The Global Frame**

The AppShell is the persistent wrapper around every screen. It hosts global nav, role context, notification access, wallet shortcut, modals, and toasts.

  ---------------------------------------------------------------------------------------------------------------------
  **Sub-component**       **Platform**        **Responsibility**
  ----------------------- ------------------- -------------------------------------------------------------------------
  \<TopBar /\>            Web only            Role switcher, search, notification bell, wallet shortcut, profile menu

  \<MiniAppTopBar /\>     Telegram only       Minimal header --- title + back button only

  \<SideNav /\>           Web desktop only    Full nav tree filtered by role + feature availability

  \<BottomNav /\>         Mobile + Telegram   Tab bar --- same nav config, filtered by role

  \<GlobalModalHost /\>   Both                Mounts all modals into a portal at root level

  \<ToastHost /\>         Both                Renders toast queue; auto-dismisses non-P0 toasts
  ---------------------------------------------------------------------------------------------------------------------

> type TopBarProps = { activeRole: Role; roles: Role\[\]; onRoleChange: (role: Role) =\> void; unreadCount: number; onOpenNotifications: () =\> void; };

## **4.3 Game Ticker (Website Header Component)**

The Game Ticker is a live, horizontally scrolling strip in the website header. It broadcasts real-time tournament activity, results, and status across the platform. Every item is clickable and links directly to the relevant bracket or tournament page.

### **Why It Exists**

-   Creates ambient awareness of platform activity --- makes the platform feel alive

-   Drives passive re-engagement: a player sees a result for a game they play and clicks through

-   Sponsor surge badges in the ticker generate impressions without requiring active browsing

-   Champion announcement state drives community sharing and virality

### **Behavior Rules**

-   Scrolls continuously left --- CSS marquee or JS-driven for control

-   Each item is a clickable chip: deep links to bracket page or tournament detail

-   \'Live Now\' items are highlighted with a pulsing green dot

-   Surge badge appears when a tournament registration is spiking

-   Champion announcement shows winner\'s gamertag + trophy icon --- stays for 60 seconds then scrolls off

-   Pauses scroll on hover/focus for accessibility

### **Game Ticker --- Execution Tracker**

  -----------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                      **Implementation Notes**
  ---------------------------------- ------------------------------------------------------------------------------------------------------
  ☐ Live scrolling game results      Polls or SSE-fed list of completed match results; auto-appends to scroll queue

  ☐ Clickable game link              Each ticker item wraps a \<Link to={\'/tournaments/\' + id}\> --- never opens blank tab

  ☐ Dynamic tournament status feed   Items tagged & filtered: Upcoming / Live / Completed --- drives badge color

  ☐ Highlight \'Live Now\'           Pulsing green dot on items with status === \'live\'; renders first in queue regardless of time order

  ☐ Surge badge in ticker            Shown when fill rate change \> threshold in last 15 min; links to tournament registration page

  ☐ Champion announcement state      Full-width ticker takeover for 60s after champion confirmed; includes gamertag + prize amount

  ☐ Deep link to bracket page        Ticker item click routes to /tournaments/:id?tab=bracket
  -----------------------------------------------------------------------------------------------------------------------------------------

## **4.4 Message Board Ticker (Website Header Component)**

The Message Board Ticker is a secondary announcement strip below the Game Ticker. It broadcasts admin messages, sponsor promotions, league updates, and org spotlights. Unlike the Game Ticker (which is data-driven), this ticker is editorially managed by AEU admins and sponsors.

### **Why It Exists**

-   Gives AEU admins a real-time broadcast channel visible across the entire platform

-   Gives sponsors a premium, non-intrusive visibility surface for messaging

-   P0 urgency styling (red background) ensures critical announcements cannot be missed

-   Countdown integration supports time-limited promotions and deadline reminders

### **Behavior Rules**

-   Messages are queued and displayed in scheduled order (publish_at / unpublish_at fields)

-   P0-styled messages (admin emergencies) override queue order and display immediately

-   Each message can optionally include a clickable link and a countdown timer

-   Sponsor messages are labeled with sponsor name and branded with their color

-   Org spotlight messages can include org logo

### **Message Board Ticker --- Execution Tracker**

  -------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                    **Implementation Notes**
  -------------------------------- ----------------------------------------------------------------------------------------------
  ☐ Admin announcements            Created via Admin Ticker Message Manager (/admin); published to all authenticated users

  ☐ Sponsor messages               Sponsor uploads message + optional link via /sponsor/campaigns; requires brand safety review

  ☐ Clickable custom link          Optional URL per message; opens in new tab with rel=\'noopener noreferrer\'

  ☐ Urgency styling (red for P0)   Messages tagged priority:P0 render with red background + white text + pulsing indicator

  ☐ League update broadcast        TO can post league updates that appear in ticker; scoped to league participants

  ☐ Org spotlight message          Org admin can queue a spotlight message; visible to all users for 24h

  ☐ Countdown integration          Optional countdown field on message --- shows live timer next to message text

  ☐ Scheduled publish/unpublish    publish_at + unpublish_at datetime fields; server-side cron removes expired messages
  -------------------------------------------------------------------------------------------------------------------------------

## **4.5 Core UI Components --- The LEGO Bricks**

These components are built first (Phase 1). Every screen is composed from them. They live in /packages/ui.

### **Structure Components**

  ---------------------------------------------------------------------------------------------------------------
  **Component**         **Props Summary**              **Behavior Notes**
  --------------------- ------------------------------ ----------------------------------------------------------
  \<PageHeader /\>      title, subtitle, actions\[\]   Title + optional subtitle + right-aligned action buttons

  \<SectionHeader /\>   title, viewAllLink?            Section label + optional \'View all\' link

  \<Card /\>            children, padding?, shadow?    Base card container --- 12--16px radius, soft shadow

  \<CardHeader /\>      title, badge?, action?         Top row inside card --- title + badge + optional action

  \<CardBody /\>        children                       Content area inside card with consistent padding

  \<CardList /\>        children                       Vertically stacked card list with gap

  \<StatTile /\>        label, value, delta?, icon?    KPI tile --- large number + label + trend delta
  ---------------------------------------------------------------------------------------------------------------

### **Action Components**

  --------------------------------------------------------------------------------------------------------
  **Component**           **When to Use**                                     **Style**
  ----------------------- --------------------------------------------------- ----------------------------
  \<PrimaryButton /\>     Main action on any screen --- one per screen        Filled accent

  \<SecondaryButton /\>   Supporting action                                   Outlined / ghost

  \<TertiaryButton /\>    Least important action (\'Not now\', \'Dismiss\')   Text only

  \<IconButton /\>        Icon-only actions (share, copy, close)              Icon + aria-label required

  \<CTAButton /\>         CTA Engine outputs --- urgency slot + countdown     Styled by PriorityTier
  --------------------------------------------------------------------------------------------------------

> type CTAButtonProps = { label: string; to: string; priority?: PriorityTier; expiresAt?: string; onClick?: () =\> void; };

### **Status Components**

  -----------------------------------------------------------------------------------------------------------------------
  **Component**       **Props**                                          **Notes**
  ------------------- -------------------------------------------------- ------------------------------------------------
  \<Badge /\>         status: string, tier?, color?                      Tournament cards, team health, membership tier

  \<ProgressBar /\>   value: 0--100, color?                              XP bar, fill rate, cap meter

  \<Countdown /\>     expiresAt: ISO, format: compact\|full, onExpire?   Ticks live; text fallback required for a11y

  \<InlineAlert /\>   priority: P0--P3, message, action?                 Color-coded inline alert strip

  \<Toast /\>         message, type: success\|error\|info, duration?     Auto-dismiss; P0 errors require manual dismiss
  -----------------------------------------------------------------------------------------------------------------------

## **4.6 Universal UI States --- Non-Negotiable**

Every single screen and panel must implement all six states. No exceptions. These are required before a component is considered \'done\'.

  --------------------------------------------------------------------------------------------------------------------------------------
  **State**           **Component**                 **What It Must Show**
  ------------------- ----------------------------- ------------------------------------------------------------------------------------
  Loading             \<Skeleton /\>                Skeleton placeholders matching the layout shape --- no spinners on primary screens

  Empty               \<EmptyState /\>              Explains what\'s empty + what user can do next (with a CTA)

  Error               \<ErrorState /\>              What went wrong + retry button + support link

  Offline             \<OfflineBanner /\>           Persistent banner --- especially critical for Mini App

  Permission denied   \<PermissionDeniedState /\>   Role-gating --- which role can do this + role switch prompt

  Feature locked      \<LockedFeatureCard /\>       Feature name, 3 benefits, price, Upgrade CTA, \'Not now\' secondary
  --------------------------------------------------------------------------------------------------------------------------------------

> type LockedFeatureCardProps = { featureName: string; reason: GateResult\[\'reason\'\]; requiredLabel: string; bullets: string\[\]; primaryCta: { label: string; to: string; }; secondaryCta?: { label: string; onClick: () =\> void; }; };

# **Section 5 --- Player Dashboard, Membership Features & Workflow**

## **5.1 CTA Layer --- NextBestActionCard & CTAFeed**

The NextBestActionCard is the single most important UI element in the product. It must render even when there are no urgent CTAs --- a fallback must always be shown.

> type NextBestActionCardProps = {
>
> cta?: CTA; // undefined → renders fallback
>
> fallback: { title: string; body?: string; to?: string; label?: string; };
>
> };

  ----------------------------------------------------------------------------------------------------------------
  **Scenario Prevented**   **P0 Title Example**                           **Revenue Without This**
  ------------------------ ---------------------------------------------- ----------------------------------------
  Player misses check-in   Check in now --- closes in 12:03               Entry fee lost; no-show

  Payout error             Payout failed --- action required              Trust erosion; churn

  Sponsor asset missing    Campaign blocked --- upload assets now         Campaign delayed; sponsor renewal risk

  Roster incomplete        Roster locks in 45 min --- 2 players missing   Forfeit; refund demand

  W-9 blocking payout      Submit W-9 to receive your payout              Funds held; player frustration; churn
  ----------------------------------------------------------------------------------------------------------------

## **5.2 Player Dashboard Layout**

### **Above the Fold**

-   \<NextBestActionCard /\> --- P0 takes full visual priority

-   \<PlayerDashboardHeader /\> --- rank tier, XP progress bar, streak flame, season tier

-   \<MyTournamentsPanel /\> --- active tournament cards with contextual status CTAs

> type PlayerDashboardHeaderProps = { displayName: string; rankTier: string; xp: number; xpToNext: number; streakDays: number; seasonTier: string; };

## **5.3 Player Membership --- Free vs Gold Feature Matrix**

Every feature in the membership matrix must be surfaced in the UI --- either as a visible feature (Gold) or as a gated prompt (Free). Nothing should be silently hidden.

### **Core Account (All Users)**

  ------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                              **Free**   **Gold**   **Implementation Notes**
  ------------------------------------------ ---------- ---------- -------------------------------------------------------------
  ☐ Profile creation + gamertag management   ✓          ✓          Stepper on onboarding; editable in /profile

  ☐ Games primary/secondary selection        ✓          ✓          Multi-select game picker; drives tournament filter defaults

  ☐ Profile completion meter                 ✓          ✓          Progress bar in player header; 100% = bonus XP

  ☐ Leaderboard view access                  ✓          ✓          Read-only for Free; ranked placement for Gold
  ------------------------------------------------------------------------------------------------------------------------------

### **Free Player UI**

  -------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                       **Behavior**                                  **Gate if Applicable**
  ----------------------------------- --------------------------------------------- ---------------------------------------------
  ☐ Join public tournaments           Standard entry fee; wallet + token checkout   No gate --- core free feature

  ☐ Pay standard entry fees           Hybrid UI: \$10 cash OR \$8 + tokens          No gate

  ☐ View-only leaderboards            Can view but rank placement requires Gold     LockedFeatureCard on rank slot

  ☐ Join team only (cannot create)    Can join via invite or discovery              \'Create Team\' gated --- LockedFeatureCard

  ☐ Upsell banner to Gold             Non-invasive persistent banner on dashboard   P3 --- always visible for free users

  ☐ Gate member-only tournaments      Locked card on tournament detail              LockedFeatureCard: \'Gold members only\'

  ☐ Gate ranked matchmaking           Panel visible but locked                      LockedFeatureCard with Gold upgrade CTA

  ☐ Gate featured match eligibility   Badge not shown --- gate inline               LockedFeatureCard inline

  ☐ Gate advanced stats export        Stats panel visible; export button gated      LockedFeatureCard on Export button
  -------------------------------------------------------------------------------------------------------------------------------

### **Gold Player UI**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                         **Behavior**                                                          **Revenue Link**
  ------------------------------------- --------------------------------------------------------------------- ----------------------------------------------
  ☐ Monthly free entry counter          Badge + counter in player header; decrements on each free entry use   Core Gold value prop --- drives subscription

  ☐ Member-only tournaments visible     Unlocked tournament cards in browse feed                              Gold exclusivity → retention

  ☐ League participation access         League tab visible and accessible in navigation                       Recurring engagement → frequency

  ☐ Ranked matchmaking panel            Panel unlocked in player dashboard                                    Competitive engagement → retention

  ☐ Featured match eligibility badge    Badge visible on player card in match rooms                           Status signal → retention

  ☐ Advanced stats dashboard + export   Full stats panel + CSV/PDF export button                              Power user value → retention

  ☐ Ranked leaderboard placement        Player appears ranked (not anonymous) on leaderboard                  Prestige → retention + referral

  ☐ One team creation per game          \'Create Team\' button active (not gated)                             Team = recurring entry fees

  ☐ Subscription renewal countdown      Countdown card in dashboard when \<14 days to renewal                 P2 --- protects MRR

  ☐ Upgrade/downgrade management        /settings/membership --- tier selector + change flow                  Self-serve churn management
  ----------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.4 Wallet & Compliance UI**

The wallet is trust-critical. Any confusion here causes churn. Every state must be handled clearly.

### **Wallet Features --- Execution Tracker**

  --------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                                **Panel**                     **Implementation Notes**
  -------------------------------------------- ----------------------------- -----------------------------------------------------------------------------
  ☐ USD wallet panel                           Fiat Wallet                   Balance, pending, withdraw + add funds buttons

  ☐ Token wallet panel                         Token Wallet                  Balance, weekly earned, pending drops, cashback

  ☐ Hybrid entry UI (\$10 or \$8 + tokens)     Tournament checkout           Toggle between full cash and hybrid token entry; calculates difference live

  ☐ Withdrawal request flow                    /wallet/withdraw              Amount, destination, fee preview, submit

  ☐ W-9 warning at \$550                       Fiat Wallet + withdraw flow   Warning banner at \$550; hard block on withdraw above \$600 until submitted

  ☐ KYC status banner                          Wallet + profile              Status chip: Unverified / Pending / Verified; links to KYC flow

  ☐ Withdrawal friction bonus (Gold benefit)   Withdraw flow                 Gold users see reduced friction messaging + faster processing badge

  ☐ Partial refund in tokens UI                Post-dispute / cancellation   Refund card shows token amount credited + option to use toward next entry

  ☐ Re-entry bonus banner                      Post-match results screen     \'Use your winnings for the next tournament\' CTA; token + cash options
  --------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.5 Player Workflow --- 13-Step UI State Map**

Every step in the player workflow must have a defined UI state. No step should be a dead end.

  --------------------------------------------------------------------------------------------------------------------------------------------------------
  **Step**   **Screen / Component**          **☐ UI State**                                                                    **Priority**
  ---------- ------------------------------- --------------------------------------------------------------------------------- ---------------------------
  1          ☐ Discovery page                /tournaments --- sponsor placements visible in sidebar/banner slots               P3

  2          ☐ Registration flow             /tournaments/:id --- wallet + token entry checkout stepper                        P1

  3          ☐ Checkout confirmation         Modal --- order summary + entry confirmation card                                 P1

  4          ☐ Check-in flow                 /match/:matchId --- countdown + large confirm + success state                     P0

  5          ☐ Match room panel              /match/:matchId --- opponent info, room code, stream overlay link                 P0

  6          ☐ Score submit interface        /match/:matchId --- result form: Win/Loss/Draw + screenshot upload                P0

  7          ☐ Dispute timeline UI           /match/:matchId --- dispute form + status tracker + resolution steps              P0

  8          ☐ Streaming overlay clickable   Match room --- clickable overlay link opens stream in new tab; sponsor branded    P1

  9          ☐ Post-match rewards screen     /match/:matchId --- XP gained, badge unlocked, tokens credited                    P1

  10         ☐ XP gain animation             Post-match screen --- XP bar fills with animation (web); simplified on Telegram   P2

  11         ☐ Badge unlock animation        Post-match screen --- badge reveal animation + collection grid update             P2

  12         ☐ Winnings credited UI          Wallet panel update + post-match card: \'Your winnings have been credited\'       P1

  13         ☐ Withdraw or reuse modal       Post-match --- modal: \'Withdraw to bank\' OR \'Re-enter a tournament\'           P2
  --------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.6 Gamification & Social --- Player**

Gamification drives the retention and frequency loops. Every element must be visible and dynamic.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                      **Behavior**                                                                               **Revenue Loop**
  ---------------------------------- ------------------------------------------------------------------------------------------ --------------------------------------------
  ☐ XP bar                           Fills on match completion, badge unlock, profile completion; shows progress to next rank   Rank up → more entries → entry fee revenue

  ☐ Season progress arc              Circular arc showing season XP progress; resets each season                                Seasonal engagement → retention

  ☐ Rank badge tier animation        Animation on rank-up event; new badge displayed prominently for 24h                        Prestige → sharing → acquisition

  ☐ Badge collection grid            Grid of earned badges with rarity color system; locked badges shown grayed                 Completion drive → retention

  ☐ Rarity color system              Common (gray), Rare (blue), Epic (purple), Legendary (gold)                                Visual prestige → engagement

  ☐ Streak multiplier indicator      Flame icon + multiplier number on XP bar; broken streak shows recovery CTA                 Daily habit → frequency

  ☐ Token leaderboard                Ranked list by token balance; links to player profiles                                     Social competition → spending

  ☐ Match highlight share            Share button on match result card; generates shareable image/link                          Viral acquisition

  ☐ Momentum Surge animation         Banner/overlay when surge is active in a tournament the player is in                       FOMO → entry → entry fee revenue

  ☐ Predictions pool visualization   Progress bar showing prediction pool split; taps token spending                            Token utility → spending
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 6 --- Tournament Organizer Dashboard, Membership & Ops**

## **6.1 TO Dashboard Overview**

The TO dashboard is a real-time event operations console. It behaves like a task inbox: every live event generates ops items that must be actioned before they escalate. The TO\'s primary goal is running events smoothly --- the UI must eliminate the need to chase information manually.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------
  **Panel**                **Content**                                                                      **Revenue Impact**
  ------------------------ -------------------------------------------------------------------------------- ------------------------------------------------
  NextBestActionCard       Highest-priority ops item: disputes, payout blockers, check-in risk              Prevents direct revenue loss

  LiveOpsQueue             4 count tiles: pending check-ins, disputes, waitlist, payout blockers            Real-time ops control

  EventPipelineList        Active events: fill rate bar, status badge, share/boost action                   Improves fill → more entry revenue

  Event Creation Panel     Create new event with inline feature locks                                       Feature locks drive upgrades

  Reputation / KPI Panel   Score 0--100: fill rate, dispute rate, satisfaction, payout speed, repeat rate   High score → sponsor bids + featured placement
  ----------------------------------------------------------------------------------------------------------------------------------------------------------

> type LiveOpsQueueProps = { pendingCheckins: number; openDisputes: number; waitlistCount: number; payoutBlockers: number; links: { checkins: string; disputes: string; waitlist: string; payouts: string; }; };
>
> *Any non-zero count in \'openDisputes\' or \'payoutBlockers\' must bubble as a P0 CTA candidate. These are direct revenue risks.*

## **6.2 TO Membership --- Free vs Gold vs Platinum Feature Matrix**

### **Free TO**

  -------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                     **Behavior**                                                    **Gate Condition**
  --------------------------------- --------------------------------------------------------------- -----------------------------------------------
  ☐ Host 1 tournament at a time     Single active event enforced; \'Create event\' gated beyond 1   LockedFeatureCard: \'Host up to 4 with Gold\'

  ☐ Standard bracket formats only   Single/double elim available; advanced formats locked           LockedFeatureCard on bracket format selector

  ☐ \<\$500 prize limit enforced    Prize input validation; error if \>\$500                        Inline validation error + upgrade CTA

  ☐ Basic ops console               Minimal ops panel --- check-in list + result confirm only       Full queue locked --- LockedFeatureCard
  -------------------------------------------------------------------------------------------------------------------------------------------------

### **Gold TO**

  ---------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                        **Behavior**                                                **Revenue Link**
  ------------------------------------ ----------------------------------------------------------- --------------------------------------------
  ☐ Host up to 4 tournaments           Event counter in dashboard; \'Create\' gated beyond 4       Volume → entry fee revenue

  ☐ Entry fee enablement               Fee input unlocked in event creation wizard                 Core TO monetization

  ☐ League hosting module              League tab visible in TO dashboard + creation flow          Recurring player entry → recurring revenue

  ☐ Branded brackets (logo + colors)   Logo + color picker in event creation wizard                Brand value → premium TO retention

  ☐ Priority tournament visibility     \'Priority\' badge on event card in browse feed             Fill rate → entry fee → TO revenue

  ☐ Sponsored tournament designation   \'Sponsored\' toggle in event creation                      Sponsor matching → AEU platform revenue

  ☐ 2 sponsor clickable slots          Sponsor slot manager in event settings --- 2 slots active   Sponsor revenue

  ☐ Data exporting                     Export button on ops console + analytics                    Operational value → TO retention

  ☐ Prize cap \<\$2500 enforcement     Validation in event creation; error if \>\$2500             Compliance gate
  ---------------------------------------------------------------------------------------------------------------------------------------------

### **Platinum TO**

  ----------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                               **Behavior**                                                  **Revenue Link**
  ------------------------------------------- ------------------------------------------------------------- ------------------------------------
  ☐ Unlimited tournaments                     No counter limit on events created                            Max volume → max entry revenue

  ☐ Priority sponsor placement                \'Priority match\' flag in sponsor bid panel                  Premium sponsor matching

  ☐ 4+ sponsor slots                          Slot manager expanded to 4+ active slots                      More sponsor revenue per event

  ☐ Advanced analytics dashboard              Full analytics with export, filters, benchmarks               Operational stickiness → retention

  ☐ Featured placement highlight              \'Featured\' badge in browse feed + algorithm boost           Fill rate + platform visibility

  ☐ Approval request UI for \>\$2500 prizes   Approval workflow appears automatically when prize \>\$2500   Compliance + trust

  ☐ Dedicated account manager indicator       Badge + contact card on TO dashboard                          Premium retention signal
  ----------------------------------------------------------------------------------------------------------------------------------------------

## **6.3 TO Operational UI --- Full Execution Tracker**

  -------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                        **Implementation Notes**
  ------------------------------------ ------------------------------------------------------------------------------------------------
  ☐ Event creation wizard              Multi-step stepper: name, game, format, schedule, prizes, sponsor slots, payout setup

  ☐ Sponsor slot pricing tool          Input per slot: CPM / flat fee / token-based; preview of placement on bracket page

  ☐ Escrow funding module              Payout setup gate: funds must be in escrow before event goes live; status indicator

  ☐ Ops queue panel (ops_queue)        Task inbox: each item has title, severity badge, timestamp, action button; sorted by priority

  ☐ Reputation score breakdown         Gauge 0--100 with sub-scores: fill rate, dispute rate, satisfaction, payout speed, repeat rate

  ☐ Reliability streak indicator       Badge showing consecutive events with no disputes + no payout failures

  ☐ Fill rate live progress            Progress bar on each event card showing registered/cap; updates in real-time

  ☐ Momentum Surge activation toggle   Toggle in event settings; activates surge badge + ticker boost when fill rate spikes

  ☐ Sponsor bid review panel           Incoming sponsor bids for event slots; accept/reject with preview of placement

  ☐ Payout setup compliance gate       Event cannot go live until payout method + escrow confirmed; P0 blocker shown

  ☐ Payout - crowd funding option      Users can add to prize pools via cash or tokens
  -------------------------------------------------------------------------------------------------------------------------------------

## **6.4 TO Gamification --- Execution Tracker**

  --------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                **Behavior**                                                            **Revenue Loop**
  ---------------------------- ----------------------------------------------------------------------- ---------------------------------------------------
  ☐ Reliability streak badge   Badge increments each event with no disputes + on-time payout           Streak protection → TO retention

  ☐ Sold-out badge counter     Badge shows total number of events that reached 100% fill               Status signal → attracts more sponsors

  ☐ Reputation gauge           Animated 0--100 gauge in TO dashboard + breakdown panel                 High rep → sponsor bids + featured → more revenue

  ☐ Sponsor-ready badge        Awarded when rep score \> threshold; visible to sponsors browsing TOs   Sponsor matching → AEU platform revenue
  --------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 7 --- Organization Dashboard, Membership & Ops**

## **7.1 Org Admin Dashboard Overview**

The org admin dashboard is a program operations console. Without it, an admin managing 8+ teams must check each individually --- a broken workflow causing missed check-ins and direct revenue loss. This dashboard surfaces everything at a glance.

  ---------------------------------------------------------------------------------------------------------------------------------------------
  **Panel**                  **Content**                                                                   **Revenue Impact**
  -------------------------- ----------------------------------------------------------------------------- ------------------------------------
  Org Header                 Logo, tier badge, org rank                                                    Identity + status at a glance

  TeamHealthPanel            Grid of team cards: status color, check-in %, disputes, roster completeness   Core ops view --- prevents forfeit

  OrgCapMeter                Seats + teams used vs cap with upgrade CTA                                    Upgrade revenue trigger

  Bulk Registration Center   Register multiple teams to a tournament at once                               Reduces friction → more entries

  Org Wallet + Budget        Total budget + per-team allocations (web-first)                               Sponsor/budget tracking

  Affiliate by Team          Referral performance per team                                                 Referral acquisition revenue
  ---------------------------------------------------------------------------------------------------------------------------------------------

## **7.2 Org Membership --- Gold vs Platinum Feature Matrix**

### **Gold Org**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                            **Behavior**                                                                     **Revenue Link**
  ---------------------------------------- -------------------------------------------------------------------------------- ---------------------------------------------
  ☐ 4 teams cap                            Team counter in dashboard; \'Create Team\' gated beyond 4                        Cap meter → upgrade revenue

  ☐ 50 seats cap                           Seat usage meter; add seat gated beyond 50                                       Cap meter → upgrade revenue

  ☐ Multi-admin (3)                        Admin management panel in /org; up to 3 admin seats                              Operational stickiness

  ☐ Finance role                           Finance-only admin role: wallet + billing access, no roster access               Separation of duties → enterprise readiness

  ☐ Org wallet                             Centralized org balance for entry fees + team allocation                         Bulk spending → entry revenue

  ☐ Budget allocation per team             Allocate org wallet funds to each team; team uses allocated budget for entries   Financial control → org retention

  ☐ Bulk tournament registration           Select tournament + select teams → register all at once                          Reduced friction → more entries

  ☐ Bulk invites (email/Telegram import)   CSV or Telegram username import for player invites                               Faster team formation

  ☐ Player Premium seat bundles            Purchase Gold memberships for players in bulk from org wallet                    Org subsidizes player membership → Gold MRR

  ☐ Org-only leagues                       Org can participate in org-tier leagues not visible to solo players              Exclusivity → org retention

  ☐ Standard analytics dashboard           Team performance, entry history, wallet activity                                 Operational value

  ☐ Affiliate dashboard                    Referral links per team + conversion tracking                                    Referral revenue

  ☐ Marketplace storefront (basic)         Basic org storefront page in marketplace                                         GMV revenue

  ☐ Prize split workflow                   Set prize distribution % across team members; auto-credited on payout            Org management value
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------

### **Platinum Org**

  ------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                        **Behavior**                                                             **Revenue Link**
  ------------------------------------ ------------------------------------------------------------------------ ----------------------------------------
  ☐ Unlimited teams                    No counter limit                                                         Max volume → max entry revenue

  ☐ Unlimited seats                    No seat limit                                                            Max scale → max player revenue

  ☐ Unlimited admins                   Admin panel: no cap on admin seats                                       Enterprise readiness

  ☐ Featured org placement             \'Featured\' badge in org directory + algorithm boost                    Acquisition → more players joining org

  ☐ Advanced analytics + export        Full analytics + CSV/PDF export                                          Operational stickiness

  ☐ Premium storefront customization   Custom banner, color theme, featured products in marketplace             GMV increase

  ☐ Priority sponsor matchmaking       Org flagged for priority in sponsor matching algorithm                   Sponsor revenue → AEU platform revenue

  ☐ Dedicated account handling         Account manager badge + contact card on org dashboard                    Premium retention signal

  ☐ API/integration placeholder UI     Settings panel showing \'API Access\' as coming soon + waitlist signup   Future revenue signal

  ☐ Advanced prize split templates     Saved prize split templates; auto-apply to new events                    Operational efficiency

  ☐ Dispute priority routing           Org disputes routed to senior review queue; faster resolution            Trust → retention
  ------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.3 Org Dashboard Panels --- Execution Tracker**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Panel**                             **Implementation Notes**
  --------------------------------------- --------------------------------------------------------------------------------------------------------------------
  ☐ Team health panel (team_health_map)   Grid of TeamHealth cards: status color (ready/attention/at_risk), check-in %, disputes open, roster lock countdown

  ☐ Seat usage meter                      ProgressBar: seatsUsed/seatsCap; P1 at 80%, P0 at 100%; upgrade CTA inline

  ☐ Team cap usage meter                  ProgressBar: teamsUsed/teamsCap; same threshold rules as seat meter

  ☐ Org XP + rank tier                    StatTile: org XP total + current tier badge; feeds org leaderboard rank

  ☐ Leaderboard rank                      Org rank in org-tier leaderboard; links to full leaderboard page

  ☐ Token pool + stake panel              Org token balance + staking controls (see Section 10 --- Token System)

  ☐ Affiliate by team analytics           Per-team referral link performance: clicks, signups, conversions, revenue

  ☐ Recruit page preview                  Live preview of org\'s public recruit page + copy invite link CTA

  ☐ Sponsor deal history                  List of past + active sponsor deals with org; deal value + status

  ☐ GMV lifetime                          StatTile: total gross merchandise value generated through org marketplace storefront
  ------------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.4 Org Gamification --- Execution Tracker**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                 **Behavior**                                                                **Revenue Loop**
  ----------------------------- --------------------------------------------------------------------------- --------------------------------------------------
  ☐ Org XP tracker              XP accumulates from team wins, event participation, org growth milestones   Org prestige → player attraction → entry revenue

  ☐ Org rank badge              Tier badge in org header + public recruit page                              Prestige signal → recruitment

  ☐ Cross-org challenge board   Org vs org challenge board with leaderboard + prize pool                    Engagement → entry revenue

  ☐ Org leaderboard             Org tier ranking across the platform; updated weekly                        Competition → org retention

  ☐ Sponsor badge display       Badges for active sponsor relationships shown on org public page            Sponsor credibility → more deals
  ------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 8 --- Sponsor Dashboard & Membership Features**

## **8.1 Sponsor Dashboard Overview**

The sponsor dashboard is a performance marketing interface. Clear ROI visibility is the primary retention mechanism. Sponsors who can easily see the value of their placement will renew. Sponsors who cannot will churn.

  --------------------------------------------------------------------------------------------------------------------------------------
  **Panel**                  **Content**                                                        **Revenue Logic**
  -------------------------- ------------------------------------------------------------------ ----------------------------------------
  NextBestActionCard         Asset upload needed, campaign blocked, renewal due, tier upgrade   Prevents campaign failure + churn

  Campaign KPI Panel         Impressions, CTR, token redemptions, conversions                   Visible ROI → renewal + spend increase

  Creative Asset Checklist   Logo, banner, copy, stream overlay --- missing items marked red    Missing assets = P0 block

  Placement Inventory        Available vs locked slots; locked shows upgrade prompt             Upgrade revenue

  Billing / Invoices         Invoice history + upcoming charges (web-first)                     Trust + transparency → retention
  --------------------------------------------------------------------------------------------------------------------------------------

## **8.2 Sponsor Membership --- Base vs Platinum Feature Matrix**

### **Base Sponsor**

  --------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                    **Behavior**                                             **Gate Condition**
  -------------------------------- -------------------------------------------------------- --------------------------
  ☐ Marketplace access             Browse and purchase standard placement inventory         No gate --- core feature

  ☐ Self-serve checkout            Checkout flow for placements; CPM or flat fee options    No gate

  ☐ Placement inventory browser    Visual grid of available placement slots with pricing    No gate

  ☐ Standard brand safety review   Asset submission → review queue → approval before live   Mandatory

  ☐ Standard dashboard             Impressions, CTR, basic redemption stats                 No gate
  --------------------------------------------------------------------------------------------------------------------

### **Platinum Sponsor**

  ----------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                    **Behavior**                                                            **Revenue Link**
  -------------------------------- ----------------------------------------------------------------------- -------------------------------------
  ☐ Custom placements              Custom placement builder; beyond standard inventory                     Premium revenue tier

  ☐ Dedicated account manager      Badge + contact card on sponsor dashboard                               Premium retention

  ☐ Enhanced brand safety review   Priority review queue; dedicated reviewer assigned                      Premium SLA → retention

  ☐ Premium reporting tier         Full attribution + export + benchmark vs category peers                 ROI clarity → renewal

  ☐ Category exclusivity lock UI   Lock out competitor brands from category; toggle in campaign settings   Premium value → high contract value

  ☐ Naming rights toggle (MVP)     Event naming rights; sponsor name in event title + bracket header       Brand value → highest tier spend

  ☐ GMV attribution panel          Revenue attributed to sponsor\'s presence in marketplace                ROI proof → renewal

  ☐ Campaign performance score     Composite score 0--100: CTR, redemptions, conversion, satisfaction      Performance transparency → renewal

  ☐ Renewal streak badge           Badge for consecutive renewal periods                                   Prestige + loyalty signal
  ----------------------------------------------------------------------------------------------------------------------------------------------

## **8.3 Sponsor Campaign UI --- Full Execution Tracker**

  ---------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                            **Implementation Notes**
  ---------------------------------------- ----------------------------------------------------------------------------------------------------
  ☐ Token drop creator                     Campaign builder: token amount, eligibility rules, duration, activation date

  ☐ Badge creator                          Design badge: name, icon, rarity, unlock condition (purchase / entry / view)

  ☐ Stream overlay upload                  Upload overlay image/video; preview on mock stream frame; dimensions enforced

  ☐ Creative asset checklist enforcement   Checklist shown on campaign dashboard; missing required assets = P0 block; campaign cannot go live

  ☐ CPM / flat fee input                   Pricing selector in campaign creation; calculates estimated reach

  ☐ Performance campaign toggle            Toggle: Standard (flat fee) vs Performance (pay-per-redemption)

  ☐ Token pool purchase                    Token pool input: buy X tokens to distribute in campaign

  ☐ Remaining token pool meter             ProgressBar showing tokens distributed vs total pool; depletion = campaign pause

  ☐ Campaign activation switch             Manual on/off toggle; auto-off when assets missing or pool depleted

  ☐ ROI dashboard                          Impressions, CTR, redemptions, GMV attribution, performance score

  ☐ Renewal CTA                            Renewal prompt card appears 30 days before contract end; P2 priority
  ---------------------------------------------------------------------------------------------------------------------------------------------

## **8.4 Sponsor Gamification --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**            **Behavior**                                                            **Revenue Loop**
  ------------------------ ----------------------------------------------------------------------- ---------------------------------
  ☐ Category leaderboard   Sponsors ranked by GMV + redemptions within their category              Competition → spend increase

  ☐ Renewal streak badge   Badge increments each renewal period; shown on sponsor public profile   Loyalty prestige → renewal

  ☐ Launch partner badge   Awarded to early sponsors; permanent badge on profile                   Recognition → platform advocacy
  ----------------------------------------------------------------------------------------------------------------------------------

# **Section 9 --- Marketplace Layer (All Roles)**

## **9.1 Marketplace Overview**

The Marketplace is a phase-gated commerce layer that evolves over time. Phase 1 launches visibility placements and sponsored items. Phase 2 adds API routing for external products. Phase 3 adds full GMV attribution. Every marketplace interaction is tracked and contributes to the token economy.

  ------------------------------------------------------------------------------------------------------------------------------
  **Phase**   **Capability**                                                          **Revenue Type**
  ----------- ----------------------------------------------------------------------- ------------------------------------------
  Phase 1     Visibility placements, sponsored drops, event merch, basic storefront   Placement fees + token spend + merch GMV

  Phase 2     API routing placeholder for external product catalog integration        Third-party GMV rev-share

  Phase 3     Full GMV attribution display for sponsors and orgs                      Attribution data as product value
  ------------------------------------------------------------------------------------------------------------------------------

## **9.2 Player Marketplace --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                            **Implementation Notes**
  ---------------------------------------- -----------------------------------------------------------------------------------------------
  ☐ Game-specific gear feed                Product feed filtered by player\'s primary/secondary games; sponsor-sourced items

  ☐ Sponsored tournament kits              Kits bundled with tournament entry; shown on tournament detail page + checkout

  ☐ Limited event merch                    Time-limited merch drops tied to specific events; countdown on product card

  ☐ Live stream clickable overlays         Overlay item visible in match stream; click opens product in marketplace

  ☐ Post-match product suggestions         After result screen: \'Players who won this event also bought\...\' card

  ☐ \'Use winnings toward purchase\' CTA   Post-match / wallet: CTA to apply fiat winnings directly to marketplace checkout

  ☐ Token cashback meter                   Shows cashback tokens earned on purchase; updates in real-time in cart

  ☐ Token-exclusive drops                  Products only purchasable with tokens; shown with token-only badge; cash purchase gated

  ☐ Premium-only discounts                 Discount badge on items; discount applied automatically for Gold members; Free users see gate
  ----------------------------------------------------------------------------------------------------------------------------------------

## **9.3 Org Marketplace --- Execution Tracker**

  -----------------------------------------------------------------------------------------------------------------
  **☐ Feature**                   **Implementation Notes**
  ------------------------------- ---------------------------------------------------------------------------------
  ☐ Org storefront page           Public org page in marketplace; shows org branding + featured products

  ☐ Bulk apparel packages         Multi-unit apparel bundles priced for team orders; quantity selector

  ☐ Equipment kits                Peripheral kits (keyboards, mice, headsets) bundled for team purchase

  ☐ Team referral leaderboard     Referral performance by team; drives inter-team competition for org token pools

  ☐ Org-level GMV tracking        StatTile on org dashboard: total GMV generated through org storefront

  ☐ Prize-to-gear allocation UI   On payout: option to convert prize into store credit for gear purchase
  -----------------------------------------------------------------------------------------------------------------

## **9.4 Tournament Organizer Marketplace --- Execution Tracker**

  --------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                      **Implementation Notes**
  ---------------------------------- ---------------------------------------------------------------------------------
  ☐ Event merchandise storefront     TO can add event-branded merch to their event page; sold through marketplace

  ☐ Pre-event promo drops            Scheduled token or merch drop before event goes live; drives early registration

  ☐ Bracket page product placement   Product placement slots on public bracket page; sponsor or TO controlled

  ☐ Champion merch                   Auto-generated champion merch after event; winner name on item; limited run
  --------------------------------------------------------------------------------------------------------------------

## **9.5 Sponsor Phase Evolution --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                       **Phase**   **Implementation Notes**
  ----------------------------------- ----------- ----------------------------------------------------------------------------------------
  ☐ Phase 1 visibility placements     1           Standard placement inventory: banner slots, bracket sponsorship, event naming

  ☐ Phase 2 API routing placeholder   2           Settings UI: \'Connect product catalog via API\' --- coming soon state with waitlist

  ☐ Phase 3 GMV attribution display   3           GMV panel in sponsor dashboard showing sales directly attributed to sponsor placements
  ----------------------------------------------------------------------------------------------------------------------------------------

# **Section 10 --- Token System (All Utilities)**

## **10.1 Token System Overview**

The token system is a cross-role digital economy that drives engagement, spending, and retention. Tokens are earned through participation and spent on entries, marketplace items, and boosts. Every token utility must be surfaced in the appropriate role\'s UI.

> *Tokens are not a currency for external withdrawal. They are an internal engagement and spending tool. The UI must never imply tokens can be directly converted to USD --- only used within the platform.*

## **10.2 Player Token Utilities --- Execution Tracker**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Utility**                           **Where It Appears**                      **Implementation Notes**
  --------------------------------------- ----------------------------------------- -------------------------------------------------------------------------
  ☐ Hybrid entry (\$8 + tokens vs \$10)   Tournament checkout                       Toggle in checkout; calculates token cost live; validates balance

  ☐ Token-gated tournaments               Tournament browse + detail                Lock badge if insufficient tokens; shows token cost

  ☐ Performance rewards                   Post-match rewards screen                 Token amount shown in reward card; credited to wallet

  ☐ Daily login rewards                   Player dashboard --- login streak panel   Token drop on login; accumulated per streak day

  ☐ Weekly active bonus                   Player dashboard --- token panel          Bonus credited on 7th consecutive active day

  ☐ Re-entry bonus                        Post-match withdraw/reuse modal           Bonus tokens offered if player re-enters within 24h

  ☐ Profile completion bonus              Profile completion meter                  One-time token bonus on 100% profile completion

  ☐ Feedback bonus                        Post-match feedback card                  Token reward for completing event feedback form

  ☐ Referral bonus                        Affiliate panel + wallet                  Token credited when referred user completes first entry

  ☐ Token cashback                        Wallet + marketplace checkout             \% cashback on purchases; shown in cart + wallet history

  ☐ Token-exclusive flash sales           Marketplace --- token drop section        Time-limited; countdown on product card; token-only purchase

  ☐ Premium viewing unlock                Match room / stream section               Token spend unlocks premium stream access

  ☐ Token predictions (Premium)           Match room --- predictions panel          Spend tokens to predict match outcome; winner pool split

  ☐ Token tips in stream                  Match room / stream overlay               Send tokens to featured player during live stream

  ☐ Withdrawal friction bonus             Wallet --- withdraw flow (Gold)           Bonus tokens offered as incentive to keep funds on platform vs withdraw
  -----------------------------------------------------------------------------------------------------------------------------------------------------------

## **10.3 Org Token Utilities --- Execution Tracker**

  -----------------------------------------------------------------------------------------------------------------------------------
  **☐ Utility**                    **Where It Appears**                 **Implementation Notes**
  -------------------------------- ------------------------------------ -------------------------------------------------------------
  ☐ Token threshold feature gate   Org dashboard --- token pool panel   Some org features require minimum token balance to activate

  ☐ Bulk token purchase            Org wallet panel                     Purchase X tokens for org pool; distributed to teams

  ☐ Internal distribution tool     Org dashboard --- token pool panel   Allocate org tokens to specific teams or players

  ☐ Org multiplier campaigns       Token pool panel                     Activate X2 token earn for all org members during event

  ☐ Token staking UI               Org dashboard --- stake panel        Stake org tokens for bonus multiplier; lock period shown

  ☐ Sponsor boost via stake        Stake panel                          Staking org tokens unlocks sponsor matching priority boost
  -----------------------------------------------------------------------------------------------------------------------------------

## **10.4 Tournament Organizer Token Utilities --- Execution Tracker**

  ----------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Utility**                        **Where It Appears**                       **Implementation Notes**
  ------------------------------------ ------------------------------------------ --------------------------------------------------------------
  ☐ Token-gated tournament toggle      Event creation wizard                      Toggle: require token balance to register

  ☐ Token prize pool bonus             Event creation --- prize setup             Add token bonus on top of cash prize pool

  ☐ Early registration reward config   Event creation --- registration settings   Configure token bonus for players who register before X date

  ☐ Partial refund in tokens config    Event creation --- refund policy           Set % of entry fee returned as tokens on cancellation

  ☐ Token visibility boost spend       Ops console --- event card                 Spend TO\'s tokens to boost event in browse feed for X hours
  ----------------------------------------------------------------------------------------------------------------------------------------------

## **10.5 Sponsor Token Utilities --- Execution Tracker**

  -------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Utility**                      **Where It Appears**                 **Implementation Notes**
  ---------------------------------- ------------------------------------ -------------------------------------------------------------------------------
  ☐ Sponsor-funded token drop UI     Campaign builder                     Configure token drop: amount, trigger (match win / purchase / view), duration

  ☐ Token multiplier campaign        Campaign builder                     Activate X2 earn for players engaging with sponsored events

  ☐ Token badge unlock requirement   Badge creator                        Require token spend to unlock sponsor-branded badge

  ☐ Token-exclusive product sales    Marketplace --- sponsor storefront   Products available only with token purchase

  ☐ Sponsor token pool lock          Campaign dashboard                   Lock token pool size at campaign start; prevents overspend
  -------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 11 --- Notifications Center & Deep Links**

## **11.1 Notifications Page Architecture**

Three tabs, each targeting a different intent. Mixing categories causes users to miss P0 items.

  --------------------------------------------------------------------------------------------------------------------------------------
  **Tab**           **Priority Levels**   **Content Examples**                                         **Badge Driven?**
  ----------------- --------------------- ------------------------------------------------------------ ---------------------------------
  Action Required   P0, P1                Check-in open, dispute window, payout blocked, roster lock   Yes --- notification bell badge

  Updates           P1, P3                Match results, XP earned, rank change, team joins            Yes --- secondary badge

  Receipts          P2                    Entry fee paid, payout received, membership renewed          No
  --------------------------------------------------------------------------------------------------------------------------------------

> type NotificationItem = { id: string; priority: PriorityTier; title: string; body?: string; createdAt: string; to: string; kind: \'cta\'\|\'receipt\'\|\'update\'; read: boolean; };

## **11.2 Notifications --- Execution Tracker**

  ---------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                         **Implementation Notes**
  ------------------------------------- -------------------------------------------------------------------------------------------
  ☐ P0 tab (Action Required)            Renders P0 + P1 notifications; sorted by priority then time; unread dot; no dismiss on P0

  ☐ P1 tab (Updates)                    Renders P1 + P3 notifications; sorted by time; all dismissible

  ☐ P2 tab (Receipts & Compliance)      Receipts + compliance notices; no badge; sorted by time

  ☐ Notification preference toggles     Settings: toggle which priority tiers trigger push/email/in-app per channel

  ☐ Telegram connection status          Settings: shows Telegram bot connection status; reconnect CTA if disconnected

  ☐ Email verification status display   Settings: email verified / unverified chip; resend verification CTA if unverified
  ---------------------------------------------------------------------------------------------------------------------------------

## **11.3 Deep Link Rules**

  ------------------------------------------------------------------------------------------------
  **Destination Type**        **When to Use**              **Example**
  --------------------------- ---------------------------- ---------------------------------------
  Screen route (web)          Primary navigation           → /match/123

  Mini App route (Telegram)   Telegram platform adapter    → tg://match/123 via platform adapter

  Modal overlay               Quick confirmations only     → modal:checkin-confirm
  ------------------------------------------------------------------------------------------------

# **Section 12 --- Membership Gating & Upgrade UX**

## **12.1 Upgrade Strategy**

Show value at the moment of intent --- when the user wants the feature. Hard-blocking browsing destroys engagement and never converts.

  ---------------------------------------------------------------------------------------------------------------------------
  **Entry Point**                       **Trigger Moment**                         **Priority**
  ------------------------------------- ------------------------------------------ ------------------------------------------
  Contextual gate (LockedFeatureCard)   User attempts a gated action               P1 --- intent-driven, highest conversion

  Cap limit gate                        80% or 100% of org/team cap reached        P0/P1 --- urgency-driven

  Persistent banner                     Always visible for free users              P3 --- passive, non-invasive

  Post-second-tournament entry          Free user enters their second tournament   P2 --- behavioral trigger

  Benefits activated card               Immediately after successful upgrade       Celebration --- reinforces decision
  ---------------------------------------------------------------------------------------------------------------------------

### **Example Gate Scenarios**

  ---------------------------------------------------------------------------------------------------------------------
  **Feature Attempted**        **Reason**   **Bullets Shown (max 3)**                                    **Required**
  ---------------------------- ------------ ------------------------------------------------------------ --------------
  Double-elimination bracket   membership   Run larger brackets, up to 256 players, priority placement   Gold TO

  Add 5th team to org          seat_cap     Manage 10 teams, bulk registration, org leaderboard          Org Pro

  Withdraw payout              kyc          Verify identity to withdraw, faster payouts, higher limits   KYC Verified

  Full analytics export        membership   Full campaign breakdown, export data, benchmark reports      Sponsor Pro
  ---------------------------------------------------------------------------------------------------------------------

> *After successful upgrade: show \'Benefits activated\' full-width success card on the home dashboard for 24 hours. This reinforces the decision and demonstrates immediate value.*

# **Section 13 --- Telegram Mini App Parity**

## **13.1 Platform Adapter**

> type PlatformAdapter = { platform: \'web\'\|\'telegram\'; openLink: (to: string) =\> void; haptics?: (kind: \'light\'\|\'medium\') =\> void; setHeaderColor?: (hex: string) =\> void; };
>
> *Components must never check \'if telegram\' directly. They call the platform adapter. This keeps the shared component tree clean and both platforms in sync.*

## **13.2 Telegram P0 Parity Checklist**

For every P0 event, the Telegram surface must have all four of these in place. Missing any one breaks the revenue protection loop.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Requirement**              **P0 Events Covered**                                              **Implementation Notes**
  ------------------------------ ------------------------------------------------------------------ ---------------------------------------------------------------------------
  ☐ Telegram push notification   Check-in, payout failure, dispute window, roster lock, W-9 block   Push sent via Telegram Bot API; includes deep link in message

  ☐ In-app P0 card (Mini App)    All P0 events                                                      NBA card renders with P0 takeover styling; countdown shown

  ☐ Correct deep link            All P0 events                                                      Platform adapter routes to correct Mini App route; validates on each push

  ☐ Correct role context         All P0 events                                                      Deep link includes role context param; Mini App sets activeRole on open
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **13.3 Telegram Mini App --- UI Constraint Checklist**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------
  **☐ Constraint**                    **Rule**                                                       **Implementation**
  ----------------------------------- -------------------------------------------------------------- --------------------------------------------------------
  ☐ Shallow navigation (max 2 deep)   No route more than 2 levels from a tab                         Route structure kept flat; sub-flows use BottomSheet

  ☐ No complex tables                 All data as card stacks + collapsible sections                 SimpleTable replaced with CardList on Telegram

  ☐ BottomSheet modals                All confirmations use BottomSheet --- no full-screen modals    Platform adapter swaps ModalConfirm for BottomSheet

  ☐ Compact wallet UI                 Token + fiat balance as StatTiles; no full transaction table   Pagination-only on Telegram; no virtual list

  ☐ Compact check-in UI               Single large confirm button + countdown; no extra panels       Full-screen P0 card with one action

  ☐ Compact ops console (TO)          At-risk items only; no full ops grid                           TeamHealthPanel renders only at_risk cards on Telegram
  -----------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 14 --- Admin (AEU Internal)**

## **14.1 Admin Dashboard --- Execution Tracker**

The admin dashboard is internal-only. It provides cross-role visibility, platform health monitoring, and editorial controls. Access restricted to aeu_admin role.

  -------------------------------------------------------------------------------------------------------------------------------------
  **☐ Feature**                    **Implementation Notes**
  -------------------------------- ----------------------------------------------------------------------------------------------------
  ☐ Cross-role filter dashboard    Filter view by role + workflow stage; see any user\'s current CTA state and dashboard context

  ☐ Revenue overview               Platform-wide revenue StatTiles: entry fees, membership MRR, marketplace GMV, token economy volume

  ☐ Affiliate payout queue         Pending affiliate payouts: amount, user, trigger event; approve/reject + bulk process

  ☐ Dispute pipeline monitor       All open disputes across platform: severity, age, assigned TO, status; escalate action

  ☐ Compliance queue (KYC + W-9)   Pending KYC verifications + W-9 submissions: review, approve, reject, request resubmit

  ☐ Churn risk monitor             Users flagged by CRM churn risk score: last login, XP trend, membership renewal date

  ☐ Token economy health           Token supply, circulating balance, burn rate, top earners, top spenders --- StatTile grid

  ☐ CTA engine health              Active P0 CTAs count, stale CTAs, completion rate, suppression hits --- monitoring panel

  ☐ Incident broadcast tool        Create system-wide P0 notification + Message Board Ticker message simultaneously

  ☐ Ticker message manager         CRUD interface for Message Board Ticker messages: create, schedule, prioritize, unpublish

  ☐ Featured live streams          Manage featured streams

  ☐ Manage tournaments             Edit all features and aspects of a tournament (minus prize money)

  ☐ Team editing                   Edit all features of team pages

  ☐ League editing                 Edit all features and aspects of a league (minus prize money)

  ☐ Leaderboard                    Edit leaderboard content and branding

  ☐ Sponsor Approvals              Managing all sponsor inquiries and sign up approvals

  ☐ Subscriptions                  Editing Features specific to memberships and pricing; Add new memberships

  ☐ User management                Managing user subscriptions; and profile fields

  ☐ News management                Editing and placement of news articles

  ☐ Action tracker                 Monitoring all actions happening on the website (view only)
  -------------------------------------------------------------------------------------------------------------------------------------

# **Section 15 --- Accessibility, Localization & Performance**

## **15.1 Accessibility Baselines**

  -------------------------------------------------------------------------------------------------------------------------------
  **Requirement**        **Standard**                                    **Notes**
  ---------------------- ----------------------------------------------- --------------------------------------------------------
  Color contrast         WCAG AA (4.5:1 body, 3:1 large text)            Check all token combinations

  Keyboard navigation    Full keyboard operability (web)                 Tab order logical; no keyboard traps

  Focus states           Visible on all interactive elements             ring utility; never remove outline without replacement

  Screen reader labels   All icons + icon-only buttons need aria-label   CTAButton, IconButton, Badge --- all labeled

  Time communication     Countdowns show text AND timer                  \'Closes in 12 minutes\' beside \'12:03\'

  Motion                 Respect prefers-reduced-motion                  All animations conditional
  -------------------------------------------------------------------------------------------------------------------------------

## **15.2 Localization Readiness**

-   All UI strings externalized via i18n keys --- no hardcoded English in components

-   Support RTL layouts eventually --- use start/end, never left/right directional values

-   Date/time rendered in user\'s local timezone from profile settings

-   Currency display respects locale formatting

## **15.3 Performance Rules**

  ------------------------------------------------------------------------------------------------------------------
  **Rule**              **Web**                                **Telegram Mini App**
  --------------------- -------------------------------------- -----------------------------------------------------
  Loading pattern       Skeleton screens for primary content   Skeleton screens --- no spinners on primary screens

  Charts                Full charts (recharts)                 KPI tiles only --- no chart libraries

  Long lists            Pagination + virtual list              Pagination only

  Bundle size           Code-split by route                    Aggressively tree-shake; target \< 200KB

  Offline               OfflineBanner + retry on reconnect     Persistent banner + last-fetched data for read-only
  ------------------------------------------------------------------------------------------------------------------

# **Section 16 --- Data Fetching, Hooks & Mutations**

## **16.1 Required Hook List**

  ----------------------------------------------------------------------------------------------------------------------------
  **Hook**                   **Returns**                                    **Notes**
  -------------------------- ---------------------------------------------- --------------------------------------------------
  useMe()                    roles, membership tier, profile completion     Used by AppShell + role switcher on every render

  useCtas({ role })          CTA\[\] sorted by priority                     Powers NextBestActionCard + CTAFeed

  useDashboard({ role })     Role-specific dashboard data bundle            Single endpoint per role reduces waterfall

  useTournamentsMine()       Tournament\[\] with status + next action       Powers MyTournamentsPanel

  useTeam(teamId)            Roster, health signals, upcoming tournaments   Powers TeamOpsPanel + TeamPage

  useOrg()                   Teams\[\], caps, budget, wallet                Powers OrgAdminHome

  useToOps()                 LiveOps counts, pipeline, reputation           Powers TOHome + LiveOpsQueue

  useSponsorCampaigns()      Campaigns\[\], assets, KPIs                    Powers SponsorHome

  useWallet()                Token balance, fiat balance, transactions      Powers all wallet panels

  useNotifications()         NotificationItem\[\] by kind                   Powers NotificationsPage

  useMarketplace({ role })   Product feed, storefront, GMV                  Powers marketplace pages
  ----------------------------------------------------------------------------------------------------------------------------

## **16.2 Required Mutations**

  ------------------------------------------------------------------------------------------------------------
  **Mutation**                     **Optimistic?**   **On Success**                     **On Error**
  -------------------------------- ----------------- ---------------------------------- ----------------------
  checkIn(matchId)                 Yes               Refetch useTournamentsMine         Revert + P0 toast

  confirmResult(matchId, result)   Yes               Refetch useTournamentsMine         Revert + error

  disputeResult(matchId, reason)   No                Refetch + dispute pending state    Error toast

  withdraw(amount, destination)    No                Refetch useWallet + show receipt   Error + support link

  upgradeMembership(tier)          No                Refetch useMe + success card       Error + retry

  approveJoinRequest(requestId)    Yes               Refetch useTeam                    Revert + toast

  activateCampaign(campaignId)     No                Refetch useSponsorCampaigns        Error + checklist

  purchaseTokens(amount)           No                Refetch useWallet                  Error + retry
  ------------------------------------------------------------------------------------------------------------

> *Optimistic updates must always revert on error with a clear toast. Never leave the UI in a stuck optimistic state.*

# **Section 17 --- Build Order & Definition of Done**

## **17.1 Phase 1 --- Foundation**

  ------------------------------------------------------------------------------------------------------------------------------------------------
  **Item**                          **Package**        **Definition of Done**
  --------------------------------- ------------------ -------------------------------------------------------------------------------------------
  Design tokens + Tailwind preset   /packages/tokens   All color, type, spacing, radius, shadow tokens defined and exported

  AppShell --- web                  /apps/web          TopBar, SideNav, BottomNav, GlobalModalHost, ToastHost working

  AppShell --- Telegram             /apps/telegram     MiniAppTopBar, BottomNav, BottomSheet host working

  Game Ticker                       /apps/web          All 7 tracker items complete; SSE or polling feed connected

  Message Board Ticker              /apps/web          All 8 tracker items complete; admin CRUD working

  Universal state components        /packages/ui       Skeleton, EmptyState, ErrorState, OfflineBanner, PermissionDeniedState, LockedFeatureCard

  CTA types + CTA Engine            /packages/core     Types, zustand store, selection logic unit tested

  NextBestActionCard                /packages/ui       P0 styling, countdown, dismiss, fallback --- all states

  Notifications center              Both apps          Three tabs, NotificationItem, deep links, read state
  ------------------------------------------------------------------------------------------------------------------------------------------------

## **17.2 Phase 2 --- Core Money Flows**

  ---------------------------------------------------------------------------------------------------------------
  **Item**                      **Route(s)**                          **Revenue Importance**
  ----------------------------- ------------------------------------- -------------------------------------------
  Tournament browse + detail    /tournaments, /tournaments/:id        Discovery + entry conversion

  Player workflow steps 1--9    Various (Section 5.5)                 Core player journey --- entry fee revenue

  Wallet pages (token + fiat)   /wallet, /wallet/withdraw             Trust + re-entry rate

  Membership upgrade surfaces   LockedFeatureCard inline + /upgrade   Subscription revenue

  Hybrid entry UI               Tournament checkout                   Token economy activation
  ---------------------------------------------------------------------------------------------------------------

## **17.3 Phase 3 --- Scale Roles**

  -------------------------------------------------------------------------------------------------
  **Item**                        **Route(s)**                          **Audience**
  ------------------------------- ------------------------------------- ---------------------------
  Captain ops panels              /home (captain view)                  team_captain

  TeamHealthPanel + OrgCapMeter   /org, /home (org view)                org_admin

  TO Ops Console                  /to/ops, /to/events                   to

  Sponsor campaigns + assets      /sponsor/campaigns, /sponsor/assets   sponsor

  Marketplace (Phase 1)           /marketplace                          All roles

  Admin dashboard                 /admin                                aeu_admin

  Token utilities (all roles)     Various --- per Section 10            All roles

  Gamification layer              Various --- per role sections         All roles
  -------------------------------------------------------------------------------------------------

## **17.4 Component Definition of Done --- Checklist**

No component is mergeable until all items below are satisfied.

  --------------------------------------------------------------------------------------------------------
  **Requirement**                  **What It Means**
  -------------------------------- -----------------------------------------------------------------------
  Props fully typed (TypeScript)   All props have explicit types. No \'any\'. No implicit any.

  Loading state                    Skeleton renders while data is fetching. No blank screen.

  Empty state                      EmptyState renders when data returns empty. Includes next-action CTA.

  Error state                      ErrorState renders on failure. Retry button + support link.

  Gated state                      LockedFeatureCard or PermissionDeniedState for gated actions.

  Mobile responsive (web)          Correct at 320px, 768px, 1024px, 1440px.

  Telegram variant                 Stacked layout, no tables, BottomSheet for confirmations.

  CTA Engine unit tests            Role filter, state filter, priority sort, expiry sort --- all tested.

  Accessibility                    aria-labels, keyboard nav, focus ring, AA contrast passing.

  Execution tracker updated        Corresponding ☐ items in this document marked ☑.
  --------------------------------------------------------------------------------------------------------

# **Section 18 --- Revenue Integration Summary**

## **18.1 The Revenue Rule**

Every UI element must serve at least one of four purposes. If it serves none, do not build it.

  ------------------------------------------------------------------------------------------------------------
  **Purpose**          **Meaning**                                  **Example UI Element**
  -------------------- -------------------------------------------- ------------------------------------------
  Protect revenue      Prevent loss of already-committed revenue    P0 check-in NBA card, payout error alert

  Drive revenue        Create or accelerate a revenue transaction   Tournament browse CTA, upgrade prompt

  Increase retention   Reduce churn; bring users back               Streak flame, XP bar, rank badge

  Increase frequency   Drive more transactions per user             Token earn/spend cycle, re-entry prompt
  ------------------------------------------------------------------------------------------------------------

> *The most expensive UI is UI that does nothing. Before building any new screen, panel, or component, identify which of the four purposes above it serves. If the answer is \'none\', don\'t ship it.*

## **18.2 Revenue Flow Map**

  -----------------------------------------------------------------------------------------------------------------------------------------
  **Funnel Stage**   **UI Surface**                           **Revenue Mechanism**                           **Risk If Missing**
  ------------------ ---------------------------------------- ----------------------------------------------- -----------------------------
  Acquisition        /welcome + onboarding stepper            Low friction = higher signups                   High drop-off

  First entry        Tournament browse + detail               Clear entry CTA = first entry fee               No conversion

  Retention          Player dashboard + streak + XP           Gamification = return visits                    Churn

  Frequency          MyTournamentsPanel + re-entry CTA        Post-result re-entry prompt                     Single-tournament players

  Monetization       Upgrade gates + cap meters               Intent-based upgrade moments                    Missed subscription revenue

  LTV                Wallet trust + payout speed              Fast payout = higher LTV                        Trust erosion + churn

  Sponsorship        Sponsor dashboard + ROI visibility       Clear ROI = renewal + spend increase            Non-renewal

  Org growth         TeamHealthPanel + OrgCapMeter            Operational value = org retention + expansion   Org churn

  Token economy      Token earn/spend surfaces across roles   Active tokens = active users = frequency        Dead token economy

  Marketplace GMV    Marketplace + post-match CTAs            Post-match spending = platform revenue          Lost commerce revenue
  -----------------------------------------------------------------------------------------------------------------------------------------

## **18.3 Figma Deliverables**

  -------------------------------------------------------------------------------------------------------------------
  **Deliverable**                      **Contents**                                               **Priority**
  ------------------------------------ ---------------------------------------------------------- -------------------
  Design tokens page                   All token values with semantic names                       P0 --- start here

  Component library                    All components in Section 4 with all states                P0

  Mobile (Mini App) frames             All screens in Telegram responsive layout                  P0

  Web responsive frames                All screens at 320px, 768px, 1440px                        P0

  Role dashboards                      Player, Captain, Org Admin, TO, Sponsor --- all 5          P0

  Game Ticker + Message Board Ticker   Both states: live data + P0 urgency styling                P0

  Clickable prototype                  Tournament register → check-in → match → result → payout   P1

  Gating UX flows                      LockedFeatureCard → upgrade → benefits activated           P1

  Token utility flows                  Hybrid entry, token drop, cashback meter                   P1

  Gamification flows                   XP gain animation, badge unlock, streak, rank-up           P2
  -------------------------------------------------------------------------------------------------------------------

*AEU UI/UX + Frontend Master Specification + Execution Tracker v3.0 \| February 2026 \| Confidential --- Internal Use Only*


---

# PART 2: CRM Backend Architecture (v3)

**AMATEUR ESPORTS UNION**

CRM Backend Architecture

*Version 3.0 \| Role-Aware + Workflow-Aware \| February 2026*

*Aligned to: AEU Frontend Master Spec v3.0 \| Audience: Engineering + Product + Executive \| Confidential*

  ----------------------------------------------------------------------------------------------------------------------------------------------------
  **Category**           **Detail**
  ---------------------- -----------------------------------------------------------------------------------------------------------------------------
  Version                CRM Architecture v3.0 --- updated from v2.0 to align with Frontend Master Spec v3.0

  Purpose                Documents all backend CRM data models, event triggers, CTA logic, and channel routing rules

  Frontend Counterpart   AEU_Front_End_Master_Spec_v3.docx --- every CRM field maps to a named FE component or hook

  Alignment Rule         Every CRM field that powers a UI panel is annotated with its FE component. Every FE hook is mapped to its CRM data source.

  New in v3              Added: ticker feeds, token system fields, gamification fields, marketplace fields, FE contract alignment columns throughout

  Classification         Confidential --- Internal Use Only --- Not for Distribution
  ----------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 1 --- Architecture Principles & Alignment Rules**

## **1.1 The Unifying Rule: Every CRM Event Must Resolve Three Questions**

Before any event fires --- check-in alert, payout trigger, membership renewal, surge activation --- the CRM must answer these three questions. The same three questions that govern the frontend screen render (Section 1 of the FE Spec) must be resolved identically at the backend level, ensuring the UI and CRM always speak the same language.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Question**                      **CRM Resolution**                                                                                                       **Frontend Counterpart (FE Spec v3)**
  --------------------------------- ------------------------------------------------------------------------------------------------------------------------ ----------------------------------------------------------------
  Who am I talking to?              Contact roles\[\]: player \| team_captain \| org_admin \| to \| sponsor \| aeu_admin                                     activeRole in zustand store → drives \<HomePage /\> routing

  Where are they in the workflow?   cta_workflow_stage tag on every event: signup \| membership \| tournament \| teams \| leagues \| marketplace \| system   WorkflowStage tag on every screen → CTA Engine workflow filter

  What should they see?             cta_priority + channel routing (Telegram push / in-app / email) + cta_primary_action deep link                           NextBestActionCard + CTAFeed + NotificationsPage tabs
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *If the CRM fires a CTA without resolving all three questions, the frontend receives an ambiguous event. The NBA card cannot render correctly, the notification routes to the wrong tab, and the deep link may fail. Resolution is mandatory on every event.*

## **1.2 Communication Priority Tiers --- CRM to Frontend Mapping**

Priority tiers govern both the CRM delivery channel and the frontend visual treatment. They must match exactly --- a P0 at the backend must render as a P0 takeover at the frontend. Mismatches cause the system to either over-alert (noise) or under-alert (missed revenue).

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Tier**   **CRM Rule + Channels**                                                                          **Frontend Behavior (FE Spec v3)**                                              **Revenue Impact**
  ---------- ------------------------------------------------------------------------------------------------ ------------------------------------------------------------------------------- ----------------------------------------------------------------------------------
  P0         Telegram Push + In-App. Time-critical. User loses something if missed. cta_expiry_at required.   Full takeover NBA card. Countdown shown. No dismiss. BottomSheet on Telegram.   Directly protects committed revenue --- check-in, payout failure, dispute window

  P1         In-App only (+ optional Telegram). Contextual. Visible when platform opened.                     Standard card in CTAFeed. Dismissible. Visible on home + relevant screen.       Drives engagement revenue --- reminders, roster alerts, badge unlocks

  P2         Email only, or Email + In-App. Transactional record.                                             Visible in Receipts tab of Notifications center only. Slim banner.              Protects subscription MRR --- renewal notices, invoices, receipts

  P3         In-App low priority. Awareness/engagement.                                                       Non-blocking card, banner, or badge in CTAFeed. Snooze allowed.                 Retention + token usage --- streaks, profile nudges, weekly recaps
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *FE alignment: The CRM cta_priority field maps directly to the PriorityTier type in /packages/core. The frontend CTAButton priority prop and NextBestActionCard styling are driven entirely by this single field.*

## **1.3 The Six Role Types --- CRM to Frontend Role Map**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Role**       **CRM Dashboard + Scope**                                                **Frontend Home Component**   **FE Hook Dependency**
  -------------- ------------------------------------------------------------------------ ----------------------------- --------------------------------------
  player         My Tournaments, wallet, rank/XP, team, affiliate, marketplace            \<PlayerHome /\>              useDashboard({ role: \'player\' })

  team_captain   Team roster, invitations, check-in compliance, match confirm/dispute     \<CaptainHome /\>             useDashboard + useTeam(teamId)

  org_admin      Multi-team ops, org wallet, bulk registration, sponsorship, affiliate    \<OrgAdminHome /\>            useDashboard + useOrg()

  to             Event pipeline, ops console, sponsor slots, payout console, reputation   \<TOHome /\>                  useDashboard + useToOps()

  sponsor        Campaign manager, placement marketplace, token campaigns, reporting      \<SponsorHome /\>             useDashboard + useSponsorCampaigns()

  aeu_admin      Cross-role health, revenue, dispute pipeline, compliance, churn risk     \<AdminHome /\>               Internal admin hooks
  ------------------------------------------------------------------------------------------------------------------------------------------------------------

## **1.4 Platform Channels --- CRM Delivery to FE Surface**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Channel**             **CRM Role**                                                     **Frontend Surface**                                                   **Critical Constraint**
  ----------------------- ---------------------------------------------------------------- ---------------------------------------------------------------------- ----------------------------------------------------
  AEU Website (Web App)   Primary account management, all roles, all tiers                 Full web app --- \<AppShell platform=\'web\' /\>                       All features available

  Telegram Mini App       Mobile-first competitive interface --- players + TOs primary     \<AppShell platform=\'telegram\' /\> --- max 2 nav depth               No tables; BottomSheet only; KPI tiles not charts

  Telegram Bot (Push)     P0 push delivery only. Requires telegram_chat_id stored.         Platform adapter: openLink() routes push deep link to Mini App route   P0 must include valid deep link or user cannot act

  Email                   Transactional only. Verification, receipts, renewal, security.   Not rendered in FE --- external email client                           Never used for P0 gameplay CTAs

  In-App Notifications    P1 + P3 tier. Both web and Mini App. Role-routed.                NotificationsPage: Action Required / Updates / Receipts tabs           Must match cta_priority to correct tab
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 2 --- Universal CRM Fields --- All User Types**

These fields apply to every AEU contact regardless of role. They power identity routing, channel delivery, CTA state tracking, and affiliate attribution. Every field listed here is consumed by the frontend via the useMe() hook in /packages/api.

## **2.1 Identity + Channel Routing Fields**

> *FE Consumer: useMe() --- returns all identity, role, and channel fields. Used by AppShell, TopBar role switcher, notification bell, and CTA Engine on every render.*

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                 **Type**         **Source**              **Notes / Logic**                                                                               **FE Component / Hook**
  ------------------------------ ---------------- ----------------------- ----------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------
  aeu_contact_id                 UUID             System                  Primary CRM record key. Assigned at account creation.                                           useMe()

  telegram_user_id               String           Telegram OAuth          Immutable Telegram ID. Primary link between Telegram identity and CRM.                          AppShell --- platform detection

  telegram_username              String           Telegram OAuth          @handle\. May be null. Fallback to display_name.                                                \<TopBar /\> profile display

  telegram_chat_id               String           Telegram bot            Required for P0 push delivery. Gate: if null, suppress Telegram channel and show connect CTA.   P0 channel suppression; connect CTA in /settings

  telegram_bot_started           Boolean          Telegram bot            True after user initiates bot. Required before any push can fire.                               Notification preferences toggle state

  telegram_miniapp_last_open     Timestamp        Mini App                Last Mini App session. Engagement signal.                                                       Churn risk monitor (admin)

  email                          String           Registration            Primary email. Transactional + lifecycle delivery.                                              useMe() --- /settings/profile

  email_verified                 Boolean          Verification flow       Gates key features. CTA trigger: resend verification.                                           \<PermissionDeniedState /\> until verified; /notifications status display

  roles\[\]                      Array (Enum)     System                  player \| team_captain \| org_admin \| to \| sponsor. Multi-role supported.                     useMe() → zustand roles\[\] → role switcher

  preferred_channel              Enum             Profile settings        telegram \| email \| in_app. Prioritizes delivery for P1/P3.                                    Notification preference toggles in /settings

  timezone                       String           Profile setup           IANA timezone. Drives accurate reminder timing (24h/1h before tournament).                      Date/time rendering in all tournament cards

  notification_opt_in            Boolean          Profile settings        Master opt-in. False = no marketing; transactional still fires.                                 Notification preferences master toggle

  notification_email_opt_in      Boolean          Profile settings        Email-specific toggle.                                                                          Notification preferences --- email row

  notification_telegram_opt_in   Boolean          Telegram bot            Telegram push toggle. Checked before every bot send.                                            Notification preferences --- Telegram row

  notification_inapp_opt_in      Boolean          Profile settings        In-app notification toggle.                                                                     Notification preferences --- in-app row

  language                       String           Profile / Telegram      ISO code. Enables localization. Defaults to \'en\'.                                             i18n token lookup in /packages/tokens

  account_status                 Enum             System                  active \| suspended \| pending_verification \| inactive                                         \<PermissionDeniedState /\> for non-active accounts

  membership_tier                Enum             Subscription            free \| gold \| platinum. Drives feature gating + CTA copy.                                     GateResult evaluation; \<LockedFeatureCard /\>

  membership_renewal_date        Date             Subscription            Drives 7-day and 1-day renewal reminder CTA triggers.                                           Subscription renewal countdown card (Gold dashboard)

  membership_status              Enum             Subscription            active \| past_due \| canceled \| paused                                                        Inline membership status badge in /settings

  created_at                     Timestamp        System                  Account creation datetime. Drives onboarding sequence timing.                                   Onboarding stepper resume logic

  last_login_at                  Timestamp        System                  Last web session. Primary churn signal.                                                         Churn risk monitor (admin)

  last_telegram_activity_at      Timestamp        Mini App                Last Mini App action. Secondary churn signal.                                                   Churn risk monitor (admin)

  profile_completion_pct         Integer 0--100   Profile system          Below 80% activates nudge sequence. 100% = completion bonus.                                    \<ProgressBar /\> in PlayerDashboardHeader; completion bonus token trigger

  kyc_status                     Enum             Identity verification   not_started \| pending \| verified \| rejected. Required for payouts.                           KYC status banner in wallet; gate on withdraw

  w9_submitted                   Boolean          Tax compliance          Triggered at \$600 cumulative winnings threshold.                                               P0 W-9 block in \<FiatWalletCard /\>; withdraw gate
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **2.2 CTA State Tracking Fields**

These fields are the backbone of the CTA Engine. Without them, the frontend cannot avoid duplicate sends, stale prompts, or incorrect priority rendering. Every CTA instance stored in the CRM maps to one CTA object in the frontend.

> *FE Consumer: useCtas({ role }) --- returns all open CTAs for activeRole. Powers NextBestActionCard + CTAFeed. The CTA type contract in /packages/core is a direct mirror of these fields.*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**         **Type**    **Source**        **Notes / Logic**                                                                                **FE Field Mapping**
  ---------------------- ----------- ----------------- ------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------------
  cta_id                 UUID        CRM task engine   Unique ID per CTA instance. One per open CTA per user.                                           CTA.id

  cta_last_seen_at       Timestamp   CRM task engine   Last time this CTA was displayed to user. Prevents re-flooding.                                  Internal CRM --- not surfaced in FE directly

  cta_state              Enum        CRM task engine   open \| completed \| dismissed \| expired. Drives suppression logic.                             CTA.state --- FE filters for state === \'open\'

  cta_priority           Enum        CRM task engine   P0 \| P1 \| P2 \| P3. Maps to channel routing + FE visual treatment.                             CTA.priority → PriorityTier → CTAButton styling

  cta_context_type       Enum        CRM task engine   tournament \| team \| membership \| affiliate \| marketplace \| wallet \| onboarding \| system   CTA.context.type --- drives which screen the deep link targets

  cta_context_id         UUID        CRM task engine   Specific entity ID (tournament_id, team_id) the CTA relates to.                                  CTA.context.id --- appended to deep link route

  cta_primary_action     String      CRM task engine   Button label + destination URL or deep link.                                                     CTA.primaryAction.{ label, to }

  cta_secondary_action   String      CRM task engine   Optional dismiss/snooze/learn_more action. Never on P0.                                          CTA.secondaryAction.{ label, kind }

  cta_expiry_at          Timestamp   CRM task engine   For time-sensitive CTAs. Auto-marks expired if missed. Required for P0.                          CTA.expiresAt → \<Countdown /\> inside NBA card

  cta_channel_sent\[\]   Array       CRM task engine   Log of channels used: \[telegram, email, in_app\]. Prevents duplicate sends.                     CTA.channelsSent\[\] --- FE reads to suppress re-show in already-sent channel

  cta_workflow_stage     Enum        CRM task engine   signup \| membership \| tournament \| teams \| leagues \| marketplace \| system                  CTA.stage --- FE CTA Engine filters by current screen\'s WorkflowStage tag

  cta_role               Enum        CRM task engine   Target role context for this CTA.                                                                CTA.role --- FE filters by activeRole before priority sort
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **2.3 Attribution + Affiliate Fields**

> *FE Consumer: Affiliate panel in \<PlayerHome /\> + org affiliate tiles in \<OrgAdminHome /\>. Fields consumed by useMe() and useOrg().*

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                **Type**   **Source**         **Notes / Logic**                                                       **FE Component**
  ----------------------------- ---------- ------------------ ----------------------------------------------------------------------- ----------------------------------------------------
  referral_code                 String     Affiliate system   Unique referral link ID. Assigned at signup to all roles.               Affiliate panel --- copy link button

  referred_by_user_id           UUID       Affiliate system   ID of referring user. Null if organic.                                  Attribution tracking --- not surfaced directly

  affiliate_link_id             String     Affiliate system   Specific link instance. Supports multi-link per user.                   Affiliate panel --- multi-link selector (future)

  affiliate_conversion_status   Enum       Affiliate system   pending \| active \| ended. Tracks lifecycle of each referred member.   Affiliate funnel visualization in player dashboard

  affiliate_months_remaining    Integer    Affiliate system   Months of recurring payout left. Max 12. Drives cap meter display.      Affiliate panel --- recurring months meter

  affiliate_conversions_count   Integer    Affiliate system   Total paid member conversions by this referrer.                         Affiliate panel --- conversion count stat

  affiliate_earnings_pending    Decimal    Affiliate system   Unpaid affiliate balance awaiting payout cycle.                         Affiliate panel --- earnings pending display

  affiliate_earnings_lifetime   Decimal    Affiliate system   All-time affiliate payouts received.                                    Affiliate panel --- lifetime earnings stat

  utm_source                    String     Attribution        Acquisition source at first visit.                                      Admin attribution dashboard

  utm_campaign                  String     Attribution        Campaign ID for growth attribution.                                     Admin attribution dashboard
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 3 --- Player Profile --- Extended Field Mapping**

Extended fields for every player account (free and Gold). Appended to universal profile. All fields are consumed by useDashboard({ role: \'player\' }) and useTournamentsMine() hooks in /packages/api.

## **3.1 Player Identity + Competitive Profile**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**         **Type**     **Source**      **Notes / Logic**                                                                  **FE Component**
  ---------------------- ------------ --------------- ---------------------------------------------------------------------------------- ----------------------------------------------------------------
  player_id              UUID         System          Player-specific profile key, links to aeu_contact_id.                              useMe() --- identity layer

  gamertag_primary       String       Profile setup   Main competitive handle. Required for tournament entry. Profile completion gate.   \<PlayerDashboardHeader /\> display name

  gamertags_secondary    JSON Array   Profile setup   Additional platform IDs: PSN, Xbox, Steam, Riot, Battle.net, etc.                  Games picker in /profile

  games_primary          Array        Profile setup   Top 1--3 games. Drives tournament recommendations + marketplace personalization.   Tournament filter defaults; marketplace feed personalization

  games_secondary        Array        Profile setup   Secondary game interest signals.                                                   Recommendation engine input

  player_xp              Integer      Gamification    Never decrements. Earned on entries, wins, challenges, check-ins, streaks.         \<ProgressBar /\> XP bar in \<PlayerDashboardHeader /\>

  player_rank_tier       Enum         Gamification    Bronze \| Silver \| Gold \| Platinum \| Elite \| Legend                            Rank badge in \<PlayerDashboardHeader /\> and tournament cards

  rank_tier_updated_at   Date         Gamification    Last tier promotion. Drives rank-up animation + social feed broadcast.             Rank-up animation trigger in post-match rewards screen

  season_xp_current      Integer      Season system   XP earned in current 90-day season.                                                Season progress arc in player dashboard

  season_id_current      String       Season system   Active season identifier.                                                          Season panel --- drives reset logic

  season_tier_current    String       Season system   Season tier derived from season XP.                                                Season tier label in player header
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.2 Tournament + Competitive History**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**             **Type**              **Source**          **Notes / Logic**                                                **FE Component**
  -------------------------- --------------------- ------------------- ---------------------------------------------------------------- --------------------------------------------------------
  tournament_entries_total   Integer               Tournament system   Lifetime entries. Key engagement metric.                         Competitive stats panel

  tournament_entries_paid    Integer               Tournament system   Paid entries only. Core revenue signal.                          Competitive stats panel + admin revenue monitor

  tournament_wins            Integer               Tournament system   All-time wins. Badge + leaderboard input.                        Competitive stats panel

  tournament_win_rate        Decimal               Computed            wins / total_entries. Dashboard stat.                            Competitive stats panel

  current_tournament_ids     Array                 Tournament system   Active registrations (registered, checked-in, live).             \<MyTournamentsPanel /\> card list

  tournament_history_ids     Array                 Tournament system   Completed events. Drives recap + re-entry suggestions.           Activity / Receipts tab; re-entry CTA trigger

  checkin_status             Enum per tournament   Tournament system   not_started \| open \| completed \| missed. Per tournament_id.   Tournament card status badge; P0 CTA trigger when open

  last_tournament_entry_at   Timestamp             Tournament system   Last entry date. Dormancy signal.                                Dormant player segment; win-back CTA trigger

  lifetime_winnings_usd      Decimal               Wallet system       Total cash winnings. W-9 threshold gate.                         Fiat wallet panel; W-9 warning at \$550

  winnings_in_wallet_usd     Decimal               Wallet system       Winnings retained in wallet (not withdrawn).                     Fiat wallet balance display; marketplace re-entry CTA

  wallet_balance_usd         Decimal               Wallet system       Total fiat balance. Includes winnings + added funds.             \<FiatWalletCard /\> balance display

  avg_placement              Decimal               Computed            Average finish position across all entries.                      Competitive stats panel
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.3 Gamification Fields** 

> *FE Consumer: \<PlayerDashboardHeader /\>, post-match rewards screen, badge collection grid, streak panel. All powered by useDashboard({ role: \'player\' }).*

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                 **Type**     **Source**     **Notes / Logic**                                                                     **FE Component**
  ------------------------------ ------------ -------------- ------------------------------------------------------------------------------------- --------------------------------------------------------------
  token_balance                  Integer      Token engine   Current spendable token balance.                                                      \<TokenWalletCard /\> --- primary balance display

  token_lifetime_earned          Integer      Token engine   All-time tokens earned. Leaderboard input.                                            Token leaderboard; competitive stats

  weekly_token_rank              Integer      Token engine   Rank in weekly token earn leaderboard. Top 10 = badge.                                Token wallet \'top earner\' badge

  pending_token_drops            Integer      Token engine   Tokens earned but not yet credited (pending settlement).                              Token wallet --- pending indicator

  token_cashback_this_week       Integer      Token engine   Cashback earned from marketplace purchases this week.                                 Token wallet --- cashback line item

  active_token_drops             JSON Array   Token engine   Sponsor-funded drops currently active: {sponsor, amount, tournament_id, expires_at}   Marketplace feed; tournament detail --- sponsored drop badge

  login_streak_current           Integer      Gamification   Consecutive daily logins. Drives multiplier.                                          Streak flame + day count in player dashboard

  login_streak_best              Integer      Gamification   All-time best streak. Badge milestone input.                                          Badge collection --- streak badges

  streak_multiplier_active       Decimal      Gamification   Current token earn multiplier from streak (e.g. 1.5x at 7 days).                      Streak multiplier indicator icon + value

  streak_recovery_window_at      Timestamp    Gamification   When streak recovery window closes after a break.                                     Streak broken CTA --- countdown to recovery window close

  badges_earned                  JSON Array   Gamification   All earned badges: {badge_id, name, rarity, earned_at}.                               Badge showcase (pinned 3) + badge collection grid

  badge_count_by_rarity          JSON         Computed       {common:n, rare:n, epic:n, legendary:n}. Rarity breakdown display.                    Badge collection grid --- rarity filter tabs

  weekly_challenge_progress      JSON         Gamification   Active weekly challenges: {challenge_id, name, progress, target, xp_reward}.          Challenges panel in player dashboard

  free_entries_cap_monthly       Integer      Subscription   Monthly free entry cap for Gold members.                                              Free entries counter (Gold only)

  free_entries_used_this_month   Integer      Subscription   Entries used against monthly cap. Resets on billing cycle.                            Free entries progress bar (Gold only)

  momentum_surge_active_for      Array        Surge engine   Tournament IDs where surge is currently active for this player.                       Momentum Surge animation in tournament cards

  prediction_pool_ids            Array        Predictions    Active prediction pools this player has entered.                                      Predictions pool visualization in match room
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.4 Token Utility Fields --- Player**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                       **Type**    **Source**          **Notes / Logic**                                                           **FE Component**
  ------------------------------------ ----------- ------------------- --------------------------------------------------------------------------- ----------------------------------------------------
  hybrid_entry_preference              Boolean     Profile settings    Player prefers hybrid entry (\$8 + tokens) over full cash. Default false.   Hybrid entry toggle default in tournament checkout

  token_gated_eligible                 Boolean     Computed            True if token balance \>= minimum for token-gated tournaments.              Token-gated tournament lock badge evaluation

  withdrawal_friction_bonus_eligible   Boolean     Subscription        True for Gold members --- bonus tokens offered to keep funds on platform.   Withdraw flow --- friction bonus offer display

  re_entry_bonus_window_at             Timestamp   Tournament system   When re-entry bonus window closes after elimination.                        Re-entry bonus CTA countdown in post-match modal

  profile_completion_bonus_claimed     Boolean     Profile system      One-time token bonus claimed at 100% profile completion.                    Profile completion meter --- bonus claimed state

  feedback_bonus_pending_ids           Array       Tournament system   Tournament IDs with unclaimed feedback token bonuses.                       Post-match feedback card --- token reward display
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.5 Website Ticker Feed Fields**

These fields power the Game Ticker and Message Board Ticker in the website header. They are broadcast events consumed by a WebSocket or SSE feed, not stored per-user profile.

> *FE Consumer: \<GameTicker /\> and \<MessageBoardTicker /\> components in website header. Polled or streamed via dedicated /api/ticker endpoint.*

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                    **Type**     **Source**          **Notes / Logic**                                                                                                           **FE Component**
  --------------------------------- ------------ ------------------- --------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------
  ticker_feed\[\]                   JSON Array   Tournament system   Stream of recent results + live events: {tournament_id, name, status, result?, surge_active, champion_gamertag?}            \<GameTicker /\> --- live scrolling items

  ticker_item_status                Enum         Tournament system   upcoming \| live \| completed \| surge \| champion --- drives badge color and ordering.                                     \<GameTicker /\> --- \'Live Now\' highlight + surge badge

  ticker_surge_threshold_pct        Decimal      Surge engine        Fill rate change threshold that triggers surge badge in ticker.                                                             \<GameTicker /\> --- surge badge visibility rule

  ticker_champion_display_seconds   Integer      Config              How long champion announcement displays before scrolling off (default 60).                                                  \<GameTicker /\> --- champion announcement timer

  ticker_messages\[\]               JSON Array   Admin / Sponsor     Message Board Ticker queue: {message_id, text, link_url?, priority, sponsor_id?, publish_at, unpublish_at, countdown_at?}   \<MessageBoardTicker /\> --- message queue

  ticker_message_priority           Enum         Admin / Sponsor     standard \| p0_urgent --- p0_urgent overrides queue order and shows red background.                                         \<MessageBoardTicker /\> --- urgency styling rule
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 4 --- Team Captain Profile --- Field Mapping**

Team Captain fields extend the player profile. A captain is always also a player --- the captain role is an overlay. These fields are consumed by useTeam(teamId) in /packages/api, which powers \<CaptainHome /\> and \<TeamOpsPanel /\>.

## **4.1 Team Identity + Roster**

> *FE Consumer: useTeam(teamId) → \<TeamOpsPanel /\>, \<JoinRequestList /\>, \<RosterPanel /\>. The captain also consumes all player fields via useDashboard({ role: \'team_captain\' }).*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                   **Type**          **Source**            **Notes / Logic**                                                                                **FE Component**
  -------------------------------- ----------------- --------------------- ------------------------------------------------------------------------------------------------ -----------------------------------------------------------
  team_id                          UUID              System                Team entity primary key.                                                                         useTeam(teamId) --- primary key for all team queries

  captain_user_id                  UUID              Team system           User ID of the captain. Links to aeu_contact_id.                                                 useMe() --- role check for captain overlay

  captain_team_name                String            Team setup            Public team name.                                                                                \<TeamHeader /\> name display

  team_game                        String            Team setup            Primary game for this team.                                                                      Team card + tournament filter

  team_roster_ids                  Array             Player system         All current player IDs on the roster.                                                            \<RosterPanel /\> --- member card list

  team_roster_size_active          Integer           Computed              Active players on roster. Compared against max for display.                                      \<TeamHeader /\> roster count vs max

  team_roster_max                  Integer           Subscription/Config   Max roster size (game-specific or subscription-gated).                                           Roster lock --- full state logic

  team_role                        Enum per player   Team system           captain \| member. Per player_id in roster.                                                      Role badge on each roster member card

  pending_join_requests            JSON Array        Team system           {player_id, gamertag, rank, games, requested_at}. Drives captain queue.                          \<JoinRequestList /\> --- approve/reject queue

  pending_invites_sent             JSON Array        Team system           {player_id, status: pending\|accepted\|declined, sent_at}. Captain-initiated outbound invites.   Pending Invites Sent panel --- cancel/resend actions

  team_tournament_ids_active       Array             Tournament system     Tournaments this team is currently registered in.                                                Active Tournaments panel on captain dashboard

  team_tournament_checkin_status   JSON              Tournament system     {tournament_id: {player_id: checked_in\|pending\|missed}}. Per-player per-tournament.            \<CheckInCompliance /\> --- color-coded per-player status

  roster_lock_events               JSON Array        Tournament system     {tournament_id, lock_at}. Per-tournament lock deadlines.                                         \<RosterLockCountdown /\> --- urgent when \<2h

  team_record_wins                 Integer           Tournament system     Team wins across all tournaments.                                                                \<TeamHeader /\> W/L record

  team_record_entries              Integer           Computed              Aggregate team tournament entries.                                                               \<TeamHeader /\> record + stats

  team_active_dispute_ids          Array             Dispute system        Open disputes involving this team. P0 CTA trigger.                                               Match Confirm/Dispute Queue --- P0 badge
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *CRM trigger rule: If team_active_dispute_ids.length \> 0 OR any player in team_tournament_checkin_status is \'missed\' within check-in window → generate P0 CTA with cta_context_type=\'team\', cta_context_id=team_id.*

## **4.2 Captain CTA Trigger Rules**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Condition**                                                       **CTA Priority**   **Title**                                                     **FE Destination**
  ------------------------------------------------------------------- ------------------ ------------------------------------------------------------- -------------------------------
  Check-in window open AND any player status=pending                  P0                 \[X\] players not checked in --- window closes in \[HH:MM\]   → /teams/:teamId?tab=checkin

  roster_lock_events\[x\].lock_at \< NOW + 2h AND roster incomplete   P0                 Roster locks in \[MM\] minutes --- \[N\] players missing      → /teams/:teamId?tab=roster

  team_active_dispute_ids.length \> 0                                 P0                 Open dispute requires your response --- \[Match ID\]          → /match/:matchId?tab=dispute

  pending_join_requests.length \> 0 AND unactioned \> 48h             P1                 \[N\] join requests waiting --- oldest \[X\]h ago             → /teams/:teamId?tab=requests

  Roster lock \< 24h but no P0 condition                              P1                 Roster locks tomorrow for \[Tournament\]                      → /teams/:teamId?tab=roster
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 5 --- Organization Admin Profile --- Field Mapping**

Org Admin is a program-level role managing multiple teams, org wallet, bulk registration, sponsorship, and affiliate at org scope. Fields consumed by useOrg() in /packages/api.

## **5.1 Org Identity + Structure**

> *FE Consumer: useOrg() → \<OrgAdminHome /\>, \<TeamHealthPanel /\>, \<OrgCapMeter /\>. Org tier naming aligned to FE Spec: Gold Org / Platinum Org (previously Starter/Pro/Elite in CRM v2).*

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                  **Type**   **Source**       **Notes / Logic**                                                                                 **FE Component**
  ------------------------------- ---------- ---------------- ------------------------------------------------------------------------------------------------- ------------------------------------------------
  org_id                          UUID       System           Org entity primary key.                                                                           useOrg() --- primary key

  org_name                        String     Org setup        Public org name.                                                                                  Org header display

  org_handle                      String     Org setup        Unique URL slug. Platform-wide unique.                                                            Public recruit page URL

  admin_user_id                   UUID       Org setup        Primary admin. Links to aeu_contact_id.                                                           Role permission check

  admin_user_ids                  Array      Org management   All admins. Role-based permissions tier. Max 3 for Gold, unlimited for Platinum.                  Admin management panel in /org

  finance_role_assigned_user_id   UUID       Permissions      User assigned finance role. Upsell if null at tier that supports it.                              Finance role panel; gate prompt if unassigned

  org_tier                        Enum       Subscription     gold \| platinum. (Updated from v2: Starter/Pro/Elite → Gold/Platinum to align with FE naming.)   Tier badge in org header; gate evaluation

  org_xp                          Integer    Gamification     Aggregate XP from all team performance.                                                           Org XP tracker StatTile

  org_rank_tier                   Enum       Gamification     Rising \| Established \| Elite Org. Recruiting signal.                                            Org rank badge in header + public recruit page

  org_leaderboard_rank            Integer    Computed         Platform-wide org ranking. Dashboard prestige.                                                    Leaderboard rank StatTile

  verification_badge_active       Boolean    Platform         Paid verification badge.                                                                          Org header badge

  public_recruit_page_enabled     Boolean    Org settings     Controls public visibility of recruit page.                                                       Recruit page preview toggle in /org/settings

  org_branding_config             JSON       Branding         {logo_url, primary_color, secondary_color, banner_url}                                            Org header logo; storefront branding
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.2 Team Management + Cap Fields**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**      **Type**   **Source**       **Notes / Logic**                                                                                                                                        **FE Component**
  ------------------- ---------- ---------------- -------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------
  team_count          Integer    Org management   Active teams. Compared to tier cap for upsell.                                                                                                           \<OrgCapMeter /\> teams bar

  team_count_cap      Integer    Subscription     Max teams: Gold=4, Platinum=unlimited.                                                                                                                   \<OrgCapMeter /\> teams cap; gate at 100%

  team_ids            Array      Team system      All team IDs under this org.                                                                                                                             \<TeamHealthPanel /\> --- card list source

  seat_count_used     Integer    Computed         Total active players across all teams.                                                                                                                   \<OrgCapMeter /\> seats bar

  seat_count_cap      Integer    Subscription     Max players: Gold=50, Platinum=unlimited.                                                                                                                \<OrgCapMeter /\> seats cap; gate at 100%

  team_health_map     JSON       Computed         {team_id: {status: ready\|attention\|at_risk, checkins:{checkedIn,total,closesAt?}, disputesOpen, rosterLockAt?, seatCap?}}. Drives team health panel.   \<TeamHealthPanel /\> --- each team card status color + signals

  roster_player_ids   Array      Player system    All player IDs across all teams.                                                                                                                         Seat count computation; bulk invite list
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *FE rule: team_health_map.status=\'at_risk\' for any team → generate P0 CTA with cta_context_type=\'team\'. On Telegram: render only at_risk team cards --- no full grid.*

  --------------------------------------------------------------------------------------------------------------------------------------------------------
  **Threshold**                                      **CRM Computation**      **CTA Generated**                                          **FE Priority**
  -------------------------------------------------- ------------------------ ---------------------------------------------------------- -----------------
  teams: seat_count_used / seat_count_cap \> 0.80    seats_at_80_pct = true   P1: \'Approaching seat limit --- \[X\] seats remaining\'   P1

  teams: seat_count_used / seat_count_cap \>= 1.00   seats_at_cap = true      P0: \'Seat limit reached --- upgrade to add more\'         P0

  teams: team_count / team_count_cap \> 0.80         teams_at_80_pct = true   P1: \'Approaching team limit --- \[X\] teams remaining\'   P1

  teams: team_count / team_count_cap \>= 1.00        teams_at_cap = true      P0: \'Team limit reached --- upgrade to continue\'         P0
  --------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.3 Org Wallet + Finance Fields**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                 **Type**   **Source**      **Notes / Logic**                                                **FE Component**
  ------------------------------ ---------- --------------- ---------------------------------------------------------------- --------------------------------------------------------------------
  org_wallet_balance_usd         Decimal    Wallet system   Org-level fiat balance for bulk entry + prize distribution.      Org wallet panel balance display

  team_budget_allocations        JSON       Wallet system   {team_id: allocated_usd}. Per-team budget tracking.              Budget allocation panel --- per-team input

  org_token_balance              Integer    Token engine    Org token pool for internal distribution + staking.              Token pool panel balance display

  org_token_stake_active         Integer    Token staking   Tokens currently staked for sponsor exposure boost.              Token stake panel --- active stake display

  prize_split_config             JSON       Prize system    Agreed split parameters per team. Must be accepted pre-signup.   Prize split workflow --- config display + player acceptance status

  marketplace_gmv_org_lifetime   Decimal    Marketplace     Total org-level marketplace GMV.                                 GMV lifetime StatTile in org dashboard
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.4 Org Token + Gamification Fields**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                 **Type**     **Source**         **Notes / Logic**                                                           **FE Component**
  ------------------------------ ------------ ------------------ --------------------------------------------------------------------------- ----------------------------------------------------
  org_token_distribution_log     JSON Array   Token engine       History of token distributions to teams/players.                            Token pool panel --- distribution history

  org_multiplier_active          Boolean      Token engine       True when org-level 2x token earn campaign is running.                      Multiplier active badge on org dashboard

  org_multiplier_expires_at      Timestamp    Token engine       When the active multiplier campaign ends.                                   Multiplier campaign countdown

  org_token_threshold_features   JSON         Feature gates      {feature_key: required_balance}. Features that require min token balance.   Inline gate on feature --- shows token requirement

  sponsor_badge_ids              Array        Sponsor system     Active sponsor relationships displayed on org public page.                  Sponsor badge display on org recruit page

  cross_org_challenge_ids        Array        Challenge system   Active cross-org challenges this org is participating in.                   Cross-org challenge board
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.5 Affiliate + Sponsor Fields**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**       **Type**   **Source**         **Notes / Logic**                                                         **FE Component**
  -------------------- ---------- ------------------ ------------------------------------------------------------------------- ---------------------------------------
  org_affiliate_code   String     Affiliate system   Org-level code. Different from individual player referral link.           Org affiliate dashboard --- copy link

  affiliate_by_team    JSON       Affiliate system   {team_id: {clicks,signups,conversions,earnings}}. Per-team performance.   Affiliate by team analytics panel

  sponsor_deal_ids     Array      Sponsor system     Active + historical sponsor agreements.                                   Sponsor deal history panel
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 6 --- Tournament Organizer Profile --- Field Mapping**

TO profile powers the Ops Console, event pipeline, reputation system, and sponsor slot management. Consumed by useToOps() in /packages/api.

## **6.1 TO Identity + Tier**

> *FE Consumer: useToOps() → \<TOHome /\>, \<LiveOpsQueue /\>, \<EventPipelineList /\>, \<ReputationPanel /\>. The ops_queue JSON field is the direct data source for \<LiveOpsQueue /\>.*

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**               **Type**   **Source**         **Notes / Logic**                                          **FE Component**
  ---------------------------- ---------- ------------------ ---------------------------------------------------------- --------------------------------------------
  to_id                        UUID       System             TO entity primary key.                                     useToOps() primary key

  to_display_name              String     TO setup           Public-facing TO brand name.                               Ops console header

  to_membership_tier           Enum       Subscription       free \| gold \| platinum. Feature access driver.           Gate evaluation for TO features

  to_tier_system               Enum       Gamification       Starter \| Community \| Verified \| Featured \| Elite TO   TO tier badge in ops console

  verified_badge_active        Boolean    Subscription       Paid verified badge.                                       TO profile badge

  featured_badge_active        Boolean    Platform-awarded   Non-purchasable. AEU awards on quality metrics.            Featured badge --- cannot be purchased

  sponsor_ready_badge_active   Boolean    Gamification       Auto-awarded when reputation + fill thresholds met.        Sponsor-ready badge on TO profile

  payout_setup_complete        Boolean    Compliance         Required before first event goes live.                     Event creation wizard --- P0 gate if false
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **6.2 Reputation + Analytics Fields**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                  **Type**         **Source**          **Notes / Logic**                                                                              **FE Component**
  ------------------------------- ---------------- ------------------- ---------------------------------------------------------------------------------------------- -------------------------------------------------------------------
  reputation_score                Decimal 0--100   Computed            Weighted: fill_rate + dispute_rate + satisfaction + payout_speed + repeat_registration_rate.   Reputation gauge in \<ReputationPanel /\>

  fill_rate_avg                   Decimal          TO analytics        Avg fill rate across last 30 events. 85%+ = benchmark.                                         Reputation score breakdown

  fill_rate_current_event         Decimal          Live event          Real-time fill % for active tournament. Drives Surge trigger.                                  Fill rate bar on event pipeline cards

  dispute_rate_avg                Decimal          TO analytics        Avg dispute % per event. Quality signal.                                                       Reputation score breakdown

  noshowrate_avg                  Decimal          TO analytics        Avg no-show rate. High = ops issue.                                                            TO analytics dashboard

  player_satisfaction_score_avg   Decimal          TO analytics        Avg post-tournament player rating (1--5).                                                      Reputation score breakdown

  repeat_registration_rate        Decimal          TO analytics        \% of players who enter multiple TO events.                                                    Reputation score breakdown --- primary quality metric

  time_to_settlement_avg_hrs      Decimal          TO analytics        Avg hours from final match to prize payout.                                                    Reputation score breakdown

  prize_payout_accuracy_rate      Decimal          TO analytics        \% payouts executed correctly + on time.                                                       Reputation score breakdown

  tournaments_hosted_total        Integer          Tournament system   Lifetime events.                                                                               TO stats StatTile

  tournaments_active              Integer          Tournament system   Currently running or published. Drives tier cap gate.                                          Event counter --- tier gate at Free=1, Gold=4, Platinum=unlimited

  total_rake_generated_usd        Decimal          Revenue             Total entry fee rake generated for AEU from TO events.                                         Admin revenue monitor
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **6.3 Ops + Live Event Fields**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**            **Type**   **Source**       **Notes / Logic**                                                                                         **FE Component**
  ------------------------- ---------- ---------------- --------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------
  ops_queue                 JSON       Computed         {pending_checkins:n, open_disputes:n, waitlist_count:n, payout_blockers:n}. Powers real-time ops panel.   \<LiveOpsQueue /\> --- four count tiles + deep links

  momentum_surge_active     Boolean    Surge engine     True when surge conditions met for active tournament.                                                     Momentum Surge activation toggle in event settings; Game Ticker surge badge

  sponsor_slots_available   Integer    Subscription     Slots by tier: Free=0, Gold=2, Platinum=4+.                                                               Sponsor slot manager in event settings

  sponsor_deal_ids          Array      Sponsor system   Active + historical sponsor deals.                                                                        Sponsor bid review panel
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> *CRM trigger rule: ops_queue.open_disputes \> 0 OR ops_queue.payout_blockers \> 0 → generate P0 CTA immediately. These are direct revenue protection events. cta_expiry_at = 24h from creation.*

## **6.4 Gamification Fields --- TO**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**               **Type**   **Source**     **Notes / Logic**                                             **FE Component**
  ---------------------------- ---------- -------------- ------------------------------------------------------------- -----------------------------------------
  soldout_badge_count          Integer    Gamification   Number of events that reached 100% fill.                      Sold-out badge counter in ops console

  reliability_streak_current   Integer    Gamification   Consecutive events with zero disputes + zero cancellations.   Reliability streak badge in ops console
  --------------------------------------------------------------------------------------------------------------------------------------------------------------

## **6.5 TO Token Utility Fields**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                  **Type**     **Source**     **Notes / Logic**                                                                 **FE Component**
  ------------------------------- ------------ -------------- --------------------------------------------------------------------------------- ------------------------------------------------------------------
  token_gated_event_ids           Array        Token engine   Events configured to require minimum token balance for entry.                     Token-gated tournament toggle state in event settings

  token_prize_pool_bonus_active   Boolean      Token engine   True when token bonus is added on top of cash prize for active events.            Token prize pool bonus display in event details

  early_reg_reward_config         JSON         Token engine   {tournament_id, token_bonus, deadline}. Early registration token reward config.   Early registration reward display in event creation

  partial_refund_token_pct        Decimal      Token engine   \% of entry fee returned as tokens on cancellation (configurable per event).      Partial refund config in event creation; refund card for players

  visibility_boost_spend_log      JSON Array   Token engine   Log of token spends on browse feed visibility boosts.                             Ops console --- event card boost action + history
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 7 --- Sponsor Profile --- Field Mapping**

Sponsor profile powers campaign management, placement inventory, token drops, and ROI reporting. Consumed by useSponsorCampaigns() in /packages/api.

## **7.1 Sponsor Identity + Approval**

> *FE Consumer: useSponsorCampaigns() → \<SponsorHome /\>, \<CampaignKpiPanel /\>, \<CreativeAssetChecklist /\>. All sponsors must be approved before campaigns go live.*

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**               **Type**   **Source**      **Notes / Logic**                                                              **FE Component**
  ---------------------------- ---------- --------------- ------------------------------------------------------------------------------ ---------------------------------------------------
  sponsor_id                   UUID       System          Sponsor entity primary key.                                                    useSponsorCampaigns() primary key

  brand_name                   String     Sponsor setup   Public-facing brand name.                                                      Sponsor dashboard header

  sponsor_type                 Enum       Approval        individual \| local_brand \| small_business \| national \| global              Sponsor profile display

  account_level                Enum       Subscription    base \| platinum_enterprise. Maps to FE: Base / Platinum Sponsor.              Gate evaluation for sponsor features

  approval_status              Enum       Review          pending \| approved \| rejected \| suspended. All sponsors must be approved.   Approval status banner; gate on campaign creation

  brand_safety_review_status   Enum       Brand safety    pending \| standard_cleared \| enhanced_cleared                                Asset checklist --- cleared state indicator

  sponsor_tier                 Enum       Gamification    Bronze \| Silver \| Gold \| Platinum                                           Tier badge + progression panel
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.2 Campaign + Performance Fields**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                **Type**         **Source**              **Notes / Logic**                                                                                         **FE Component**
  ----------------------------- ---------------- ----------------------- --------------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------
  campaign_performance_score    Decimal 0--100   Analytics               Weighted: impressions + CTR + token_redemptions + conversions.                                            Campaign performance score tile

  category                      String           Brand profile           Product category (e.g. energy_drink, peripherals, apparel). Drives category leaderboard.                  Category leaderboard rank display

  category_leaderboard_rank     Integer          Gamification            Rank within category. Drives exclusivity lock mechanic.                                                   Category leaderboard panel

  category_exclusivity_locked   Boolean          Gamification            True = Gold tier sponsor has locked category. Competitor prompt fires.                                    Category exclusivity lock UI toggle

  campaigns_active_count        Integer          Campaign system         Number of live campaigns.                                                                                 Campaign overview panel count

  campaigns_total               Integer          Campaign system         Lifetime campaigns.                                                                                       Sponsor profile stats

  creative_asset_checklist      JSON             Campaign system         {logo_uploaded, banner_uploaded, copy_uploaded, stream_overlay_uploaded, all_cleared}. Drives P0 block.   \<CreativeAssetChecklist /\> --- missing items red; P0 if campaign live-blocked

  token_pool_balance            Integer          Token engine            Current unspent sponsor token pool.                                                                       Token pool meter in campaign dashboard

  token_pool_locked_amount      Integer          Token engine            Tokens locked at campaign start. Prevents overspend.                                                      Token pool lock indicator

  token_campaigns_active        JSON Array       Token engine            Active token drop/multiplier campaigns: {campaign_id, type, amount, expires_at}.                          Token campaigns panel; active drop display

  gmv_attributed_usd            Decimal          Marketplace (Phase 3)   Revenue directly attributed to sponsor placements. Phase 3 only.                                          GMV attribution panel (Phase 3)

  total_spend_lifetime_usd      Decimal          Billing                 Total platform spend.                                                                                     Billing panel + admin revenue monitor

  current_period_spend_usd      Decimal          Billing                 Spend in current billing period.                                                                          Billing panel current period display

  renewal_streak_count          Integer          Subscription            Consecutive renewal periods. Loyalty badge trigger.                                                       Renewal streak badge; tier progression panel

  last_campaign_launched_at     Timestamp        Campaign system         Drives sponsor renewal risk segment.                                                                      Renewal CTA trigger (30 days before contract end)
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.3 Sponsor Feature Fields**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**               **Type**   **Source**      **Notes / Logic**                                                        **FE Component**
  ---------------------------- ---------- --------------- ------------------------------------------------------------------------ ------------------------------------------------------------------
  mvp_naming_rights_active     Boolean    Subscription    Platinum only. Sponsor name displayed in MVP badge on player profiles.   MVP Naming Rights panel (Platinum)

  mvp_badge_name_custom        String     Sponsor setup   Custom MVP badge name set by sponsor.                                    MVP badge preview in sponsor dashboard

  dashboard_access_tier        Enum       Subscription    standard \| premium_reporting. Gate on advanced analytics export.        Premium reporting gate; LockedFeatureCard on upgrade

  placement_inventory_access   JSON       Subscription    Available placement types for this account_level.                        Placement inventory browser --- locked slots show upgrade prompt
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.4 Sponsor CTA Trigger Rules**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Condition**                                                            **CTA Priority**   **Title**                                                  **FE Destination**
  ------------------------------------------------------------------------ ------------------ ---------------------------------------------------------- --------------------------------------
  creative_asset_checklist.all_cleared = false AND campaign start \< 48h   P0                 Campaign blocked --- upload missing assets now             → /sponsor/assets

  creative_asset_checklist.all_cleared = false (no imminent start)         P1                 Assets missing --- complete checklist to launch campaign   → /sponsor/assets

  token_pool_balance \< 10% of original pool                               P1                 Token pool nearly depleted --- campaign will pause soon    → /sponsor/campaigns/:id

  token_pool_balance = 0                                                   P0                 Token pool empty --- campaign paused                       → /sponsor/campaigns/:id

  contract end date \< 30 days                                             P2                 Campaign renewal due in \[N\] days                         → /sponsor/campaigns/:id?tab=renewal

  renewal_streak_count milestone (3, 6, 12)                                P1                 \[X\]-campaign renewal streak --- loyalty bonus unlocked   → /sponsor/campaigns/:id
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **7.5 Sponsor Token Utility Fields**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                     **Type**   **Source**     **Notes / Logic**                                                                                                     **FE Component**
  ---------------------------------- ---------- -------------- --------------------------------------------------------------------------------------------------------------------- ------------------------------------------------------
  sponsor_funded_drop_config         JSON       Token engine   {drop_id, trigger: match_win\|purchase\|view, amount, tournament_id?, duration}. Sponsor-funded drop configuration.   Token drop creator in campaign builder

  token_multiplier_campaign_active   Boolean    Token engine   True when sponsor is running a 2x earn multiplier campaign.                                                           Token multiplier campaign toggle in campaign builder

  token_badge_unlock_requirement     JSON       Token engine   {badge_id, token_spend_required}. Tokens required to unlock sponsor badge.                                            Badge creator --- token requirement field

  token_exclusive_products           Array      Marketplace    Product IDs available only with token purchase.                                                                       Marketplace --- token-only badge on products
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 8 --- Marketplace Fields (All Roles)** 

The marketplace is a phase-gated commerce layer. These fields track player/org/sponsor interaction with the marketplace and power personalized feeds, token cashback, and GMV attribution. Consumed by useMarketplace({ role }) hook.

## **8.1 Player Marketplace Fields**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                           **Type**     **Source**     **Notes / Logic**                                                                                   **FE Component**
  ---------------------------------------- ------------ -------------- --------------------------------------------------------------------------------------------------- --------------------------------------------------------
  marketplace_orders                       JSON Array   Marketplace    {order_id, product_id, amount_usd, tokens_used, cashback_tokens, status, ordered_at}                Order history in wallet Receipts tab

  marketplace_gmv_lifetime                 Decimal      Marketplace    Total player marketplace spend. High value player signal.                                           Admin --- high value player segment

  marketplace_active_drops                 JSON Array   Marketplace    {product_id, name, tokens_required, expires_at, remaining_quantity}. Token-exclusive flash sales.   Marketplace --- token drop section with countdown

  marketplace_winnings_conversion_prompt   Boolean      Computed       True immediately after prize payout credited. Drives \'use winnings toward gear\' CTA.              Post-match rewards screen; wallet CTA

  marketplace_premium_discount_eligible    Boolean      Subscription   True for Gold members. Discount applied automatically at checkout.                                  Marketplace product badge; checkout discount line item
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **8.2 Org Marketplace Fields**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                     **Type**     **Source**     **Notes / Logic**                                                                         **FE Component**
  ---------------------------------- ------------ -------------- ----------------------------------------------------------------------------------------- -------------------------------------------------
  org_storefront_enabled             Boolean      Subscription   Gold+ orgs can activate marketplace storefront.                                           Storefront activation toggle in /org/settings

  org_storefront_config              JSON         Marketplace    {banner_url, featured_products\[\], branding}. Platinum: full customization.              Org storefront page rendering

  org_gmv_current_period             Decimal      Marketplace    GMV in current billing period.                                                            Org dashboard GMV tracking tile

  prize_to_gear_allocation_pending   JSON Array   Marketplace    {player_id, prize_usd, converting_to_credit}. Players converting prize to store credit.   Prize-to-gear allocation UI in post-payout flow
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **8.3 TO Marketplace Fields**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                 **Type**     **Source**    **Notes / Logic**                                                   **FE Component**
  ------------------------------ ------------ ------------- ------------------------------------------------------------------- --------------------------------------------------
  event_merch_storefront_ids     Array        Marketplace   Event-branded merch storefronts for this TO\'s events.              Event merchandise storefront link on event page

  pre_event_drops_scheduled      JSON Array   Marketplace   {event_id, drop_id, publish_at}. Scheduled pre-event promo drops.   Pre-event promo drop scheduler in event settings

  champion_merch_generated_ids   Array        Marketplace   Auto-generated champion merch after event completion.               Champion merch notification + link post-event
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **8.4 Phase Gating State**

  ---------------------------------------------------------------------------------------------------------------------------------
  **Phase**   **Status Field**    **Value**        **FE Behavior**
  ----------- ------------------- ---------------- --------------------------------------------------------------------------------
  Phase 1     marketplace_phase   1                Full marketplace visible: placements, drops, event merch, storefronts

  Phase 2     marketplace_phase   2                API routing placeholder shown in /org and /sponsor settings as \'coming soon\'

  Phase 3     marketplace_phase   3                GMV attribution display active in sponsor + org dashboards
  ---------------------------------------------------------------------------------------------------------------------------------

# **Section 9 --- CTA Trigger Map --- All Workflow Stages**

This section defines every CRM event trigger, the CTA it generates, the channel it fires on, and its frontend destination. The frontend CTA Engine consumes this data via useCtas({ role }). Every trigger resolves the three questions from Section 1.1 before firing.

> *Column key: Priority = cta_priority field. Channel = delivery channel(s). FE Destination = cta_primary_action deep link consumed by platform adapter.*

## **9.1 Signup + Onboarding**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**            **Recipient**   **Priority**   **Channel**      **CRM Action + CTA**                                                            **FE Destination**
  ----------------------------------- --------------- -------------- ---------------- ------------------------------------------------------------------------------- -----------------------------
  User submits signup form            All roles       P3             In-App           \'Account created --- complete your profile.\' CTA: Complete profile.           /onboarding

  Email verification required         All roles       P2             Email            Send verification email. cta_state=open until verified.                         /onboarding?step=verify

  Email verified                      All roles       P3             In-App           \'Email verified.\' CTA: Continue setup.                                        /onboarding?step=next

  Welcome email Day 0                 All roles       P2             Email            Onboarding sequence Day 0. CTA: Finish profile.                                 /onboarding

  Gamertags missing (profile \<80%)   Player          P3             In-App           \'Add your game IDs to enter tournaments.\' CTA: Add IDs.                       /profile?tab=games

  Telegram bot not connected          Player / TO     P3             In-App           \'Connect Telegram for real-time alerts.\' CTA: Connect now.                    /settings?tab=notifications

  Profile \<80% at D+3                All roles       P3             In-App + Email   Nudge D+3 if profile_completion_pct \< 80. Token bonus incentive for players.   /onboarding?resume=true
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.2 Membership Lifecycle**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**          **Recipient**       **Priority**   **Channel**                 **CRM Action + CTA**                                                                         **FE Destination**
  --------------------------------- ------------------- -------------- --------------------------- -------------------------------------------------------------------------------------------- ---------------------------
  Membership purchased / upgraded   All roles           P2             In-App + Email              Benefits activated card + email receipt. cta_context_type=membership.                        /home (benefits card)

  Benefits activated display        Player / TO / Org   P3             In-App                      Show active benefit summary. \'Use benefits\' CTA.                                           /home

  Payment failed on purchase        All roles           P0             Email + In-App + Telegram   \'Payment failed --- update card.\' CTA: Update payment.                                     /settings/billing

  Renewal: 7 days out               All roles           P1             Email + In-App + Telegram   Renewal card with renewal_date + price. CTA: Manage plan.                                    /settings/membership

  Renewal: 1 day out                All roles           P1             Email + In-App + Telegram   Urgent renewal card. CTA: Manage plan.                                                       /settings/membership

  Renewal successful                All roles           P2             Email                       Receipt + confirmation. CTA: View billing.                                                   /billing

  Renewal failed / card expired     All roles           P0             Email + In-App + Telegram   cta_expiry_at = 72h. \'Keep benefits --- update card.\' CTA: Update card.                    /settings/billing

  Downgrade scheduled               All roles           P2             In-App                      \'Plan changes on \[DATE\].\' CTA: Undo / manage.                                            /settings/membership

  Membership canceled (voluntary)   All roles           P2             Email + In-App              \'Membership ended.\' CTA: Re-activate.                                                      /upgrade

  Membership ended (non-payment)    All roles           P0             Email + In-App + Telegram   \'Benefits ended due to billing.\' CTA: Fix billing.                                         /settings/billing

  Free user hits tournament #2      Player              P1             In-App                      Upsell: \'Upgrade for member-only events + free monthly entries.\' CTA: See Gold benefits.   /upgrade

  Free user views gated feature     Player / TO         P1             In-App                      Inline gate: \'This requires \[tier\].\' CTA: Upgrade. cta_context_type=membership.          /upgrade (or inline gate)
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.3 Tournament Lifecycle --- Player CTAs**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**         **Recipient**   **Priority**   **Channel**                 **CRM Action + CTA**                                                                  **FE Destination**
  -------------------------------- --------------- -------------- --------------------------- ------------------------------------------------------------------------------------- ---------------------------------
  Payment succeeded                Player          P2             Email + In-App              Email receipt. In-app: registered card. CTA: View lobby.                              /tournaments/:id

  Payment failed                   Player          P0             Email + In-App + Telegram   \'Retry payment to secure your spot.\' cta_expiry_at = spot hold window.              /tournaments/:id?checkout=retry

  Registered (free entry)          Player          P2             Email + In-App              \'You\'re registered.\' Receipt if paid. CTA: View details.                           /tournaments/:id

  Waitlisted                       Player          P1             In-App + Email              Show waitlist position. cta_expiry_at = tournament start.                             /tournaments/:id?tab=waitlist

  Waitlist promoted                Player          P0             Email + In-App + Telegram   \'A spot opened --- confirm now!\' cta_expiry_at = 30 min. CTA: Confirm entry.        /tournaments/:id?confirm=true

  Bracket published                Player          P1             Email + In-App + Telegram   \'Your bracket is live. First match: \[opponent\] at \[time\].\' CTA: View bracket.   /tournaments/:id?tab=bracket

  Reminder: 24h before             Player          P1             Email + In-App + Telegram   \'Starts tomorrow.\' CTA: View schedule.                                              /tournaments/:id

  Reminder: 1h before              Player          P1             In-App + Telegram           \'Starts in 1 hour.\' CTA: Open lobby.                                                /match/:matchId

  Check-in window opens            Player          P0             In-App + Telegram           \'Check in now or lose your spot.\' cta_expiry_at = check-in close.                   /match/:matchId?action=checkin

  Checked in successfully          Player          P1             In-App + Telegram           \'Checked in for \[Tournament\]. Good luck.\' Clears P0 check-in CTA.                 /match/:matchId

  Missed check-in warning          Player          P0             In-App + Telegram           \'\[X\] minutes left.\' cta_expiry_at = window close. CTA: Check in now.              /match/:matchId?action=checkin

  Tournament starting              Player          P0             Email + In-App + Telegram   \'Starting now.\' CTA: Go to match room.                                              /match/:matchId

  Result submitted (by opponent)   Player          P0             In-App + Telegram           \'Confirm or dispute within \[window\].\' cta_expiry_at = dispute window.             /match/:matchId?action=confirm

  Result confirmed                 Player          P1             In-App + Telegram           \'Result finalized.\' Clears dispute CTA. CTA: Next match.                            /match/:matchId

  Dispute opened                   Player + TO     P0             In-App + Telegram           \'Dispute requires review --- submit evidence.\' cta_expiry_at = evidence window.     /match/:matchId?tab=dispute

  Dispute resolved                 Player          P1             In-App + Telegram           \'Dispute resolved: \[outcome\].\' CTA: View bracket.                                 /tournaments/:id?tab=bracket

  Eliminated                       Player          P1             In-App + Telegram           \'Eliminated. GG.\' Re-entry CTA if open events match history.                        /tournaments (re-entry)

  Tournament complete              Player + TO     P1             Email + In-App + Telegram   \'Tournament complete.\' CTA: View results.                                           /tournaments/:id?tab=results

  Prize payout initiated           Winner          P1             Email + In-App + Telegram   \'Payout processing.\' CTA: Confirm payout info if setup incomplete.                  /wallet

  Prize payout completed           Winner          P1             Email + In-App + Telegram   \'Payout sent to wallet.\' Email receipt. Marketplace re-entry nudge fires.           /wallet

  Prize payout failed              Winner          P0             Email + In-App + Telegram   \'Action needed --- payout failed.\' cta_expiry_at = 7 days.                          /wallet/issue/:id

  Post-tournament feedback         Participant     P3             In-App                      Survey card with token reward incentive. CTA: Submit rating.                          /match/:matchId?tab=feedback
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.4 Tournament Lifecycle --- TO CTAs**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**           **Recipient**    **Priority**   **Channel**                 **CRM Action + CTA**                                                   **FE Destination**
  ---------------------------------- ---------------- -------------- --------------------------- ---------------------------------------------------------------------- ----------------------------------
  Tournament published               TO               P2             In-App + Email              \'Your tournament is live.\' CTA: View / share.                        /to/events/:id

  New participant joined             TO               P3             In-App                      \'New participant: \[handle\].\' Count badge on roster.                /to/events/:id?tab=roster

  Tournament at capacity             TO               P1             In-App                      \'Tournament full.\' CTA: Manage waitlist.                             /to/events/:id?tab=waitlist

  Player at risk (missed check-in)   TO               P0             In-App + Telegram           \'\[Player\] missed check-in.\' cta_expiry_at = roster lock.           /to/ops?event=:id&focus=checkins

  Dispute opened (any match)         TO               P0             In-App + Telegram           \'Dispute on Match \[ID\] --- review evidence.\' CTA: View dispute.    /to/ops?event=:id&focus=disputes

  Dispute escalated to admin         TO + AEU Admin   P1             In-App + Email              \'Dispute escalated.\' CTA: Review evidence.                           /to/ops?event=:id&focus=disputes

  Dispute resolved                   TO               P1             In-App                      \'Dispute resolved: \[outcome\].\' CTA: View bracket.                  /to/events/:id?tab=bracket

  Tournament complete                TO               P1             Email + In-App              \'Tournament complete. Payout initiated.\' CTA: View payout console.   /to/ops?tab=payouts

  Payout action needed               TO               P0             Email + In-App + Telegram   \'Payout failed for \[winner\].\' CTA: Fix payout details.             /to/ops?tab=payouts&issue=:id

  Post-tournament feedback posted    TO               P2             In-App                      \'Player satisfaction score updated.\' CTA: View reputation metrics.   /to/ops?tab=reputation
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.5 Teams CTAs**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**         **Recipient**        **Priority**   **Channel**         **CRM Action + CTA**                                                               **FE Destination**
  -------------------------------- -------------------- -------------- ------------------- ---------------------------------------------------------------------------------- ------------------------------
  Captain creates team             Captain              P3             In-App              \'Team created.\' CTA: Invite members. Creates captain view.                       /teams/:teamId?action=invite

  Invitation sent to player        Player               P1             Email + In-App      \'You\'ve been invited to join \[Team\] by \[Captain\].\' CTA: Accept / Decline.   /teams/:teamId?invite=true

  Invitation accepted              Captain              P2             In-App              \'\[Player\] joined your team.\' CTA: View roster.                                 /teams/:teamId?tab=roster

  Invitation declined              Captain              P2             In-App              \'\[Player\] declined.\' CTA: Invite others.                                       /teams/:teamId?action=invite

  Join request submitted           Captain + Managers   P1             In-App + Email      \'\[Player\] requests to join \[Team\].\' Adds to pending queue.                   /teams/:teamId?tab=requests

  Join request approved            Player               P2             Email + In-App      \'Approved --- welcome to \[Team\].\' CTA: View team.                              /teams/:teamId

  Join request rejected            Player               P2             In-App              \'Request not approved.\' CTA: Browse teams.                                       /teams

  Player removed from team         Player               P2             In-App + Email      \'Removed from \[Team\].\' CTA: Find new team.                                     /teams

  Roster lock approaching (\<2h)   Captain              P0             In-App + Telegram   \'Roster locks in \[X\] minutes.\' cta_expiry_at = lock time.                      /teams/:teamId?tab=roster

  Team disbanded                   All members          P1             In-App + Email      \'\[Team\] has been disbanded.\' CTA: Browse teams.                                /teams
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.6 Leagues CTAs**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**        **Recipient**      **Priority**   **Channel**                 **CRM Action + CTA**                                  **FE Destination**
  ------------------------------- ------------------ -------------- --------------------------- ----------------------------------------------------- ----------------------------------
  League published                League Admin       P2             Email + In-App              \'League is live.\' CTA: Share link.                  /leagues/:leagueId

  League signup complete          Player / Team      P2             Email + In-App              Receipt + registration confirmed. CTA: View league.   /leagues/:leagueId

  Schedule published              Participants       P1             Email + In-App + Telegram   \'Schedule available.\' CTA: View fixtures.           /leagues/:leagueId?tab=schedule

  League event reminder: 24h      Participants       P1             Email + In-App + Telegram   \'Event tomorrow.\' CTA: View details.                /leagues/:leagueId?tab=schedule

  Standings updated               All participants   P1             In-App + Telegram           \'Standings updated.\' CTA: View table.               /leagues/:leagueId?tab=standings

  Playoff qualification locked    Qualified          P1             In-App + Telegram           \'You qualified for playoffs.\' CTA: View playoffs.   /leagues/:leagueId?tab=playoffs

  League concluded                All + Admin        P1             Email + In-App              \'League complete.\' CTA: View champions.             /leagues/:leagueId?tab=results

  League prize payout initiated   Winners            P1             Email + In-App              \'Payout processing.\' CTA: Confirm payout info.      /wallet
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.7 Sponsor + Marketplace CTAs**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**                 **Recipient**      **Priority**   **Channel**                 **CRM Action + CTA**                                                              **FE Destination**
  ---------------------------------------- ------------------ -------------- --------------------------- --------------------------------------------------------------------------------- ----------------------------------------------------------------
  Sponsor account created                  Sponsor            P3             In-App                      \'Profile created.\' CTA: Complete profile.                                       /sponsor/setup

  Bid submitted on placement               Sponsor + TO       P2             In-App + Email              Sponsor: \'Bid submitted.\' TO: \'New bid received.\' CTA: Review bid.            /to/events/:id?tab=sponsors (TO); /sponsor/campaigns (Sponsor)

  TO accepts bid                           Sponsor + TO       P2             Email + In-App              \'Bid accepted.\' CTA: Upload assets. creative_asset_checklist drives next CTA.   /sponsor/assets

  TO rejects bid                           Sponsor            P2             In-App                      \'Bid not accepted.\' CTA: Browse placements.                                     /sponsor/campaigns/new

  Placement live                           Sponsor + TO       P2             In-App                      \'Campaign live.\' CTA: View placement + impressions.                             /sponsor/campaigns/:id

  Assets missing + campaign imminent       Sponsor            P0             Email + In-App + Telegram   \'Campaign blocked --- upload assets now.\' cta_expiry_at = 48h.                  /sponsor/assets

  Campaign ended                           Sponsor + TO       P2             Email + In-App              \'Campaign ended.\' CTA: View summary + renew.                                    /sponsor/campaigns/:id?tab=renewal

  Renewal due in 30 days                   Sponsor            P2             Email + In-App              \'Contract renewal due in \[N\] days.\' CTA: Review.                              /sponsor/campaigns/:id?tab=renewal

  Token pool depleted                      Sponsor            P0             Email + In-App + Telegram   \'Token pool empty --- campaign paused.\' CTA: Add tokens.                        /sponsor/campaigns/:id

  Marketplace product purchased            Player             P1             In-App + Telegram           \'Order confirmed. +\[X\] tokens cashback.\' CTA: View order.                     /wallet?tab=receipts

  Limited drop available                   Eligible players   P1             In-App + Telegram           \'Limited drop: \[Item\] --- \[X\] tokens or \[Y\] USD.\' CTA: Shop now.          /marketplace?drop=:id

  Winnings credited + marketplace prompt   Winner             P2             In-App                      \'Apply winnings toward gear for a token bonus.\' CTA: Shop now.                  /marketplace
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.8 Token + Gamification CTAs**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**            **Recipient**        **Priority**   **Channel**         **CRM Action + CTA**                                                                                   **FE Destination**
  ----------------------------------- -------------------- -------------- ------------------- ------------------------------------------------------------------------------------------------------ ---------------------
  Streak milestone: 7 days            Player               P1             In-App + Telegram   \'7-day streak --- 1.5x token earn unlocked.\' CTA: Enter tournament.                                  /tournaments

  Streak broken                       Player               P1             In-App + Telegram   \'Streak broken. Recovery window open for \[X\] hours.\' CTA: Log in.                                  /home

  Rank tier upgraded                  Player               P1             In-App + Telegram   \'Level up --- you\'re now \[Tier\].\' Broadcast to followers.                                         /profile

  Badge earned                        Player               P1             In-App + Telegram   \'New badge: \[Name\] (\[Rarity\]).\' CTA: View badge collection.                                      /profile?tab=badges

  Season complete (top tier)          Player               P2             Email + In-App      \'Season complete. Bonus: \[X\] tokens.\' CTA: View season rewards.                                    /profile?tab=season

  Momentum Surge activates            All active players   P0             In-App + Telegram   \'Surge live on \[Tournament\]: 2x tokens for next \[N\] registrations.\' cta_expiry_at = surge end.   /tournaments/:id

  Token drop (sponsor-funded)         Eligible players     P1             In-App + Telegram   \'\[Brand\] dropping \[X\] tokens --- earn by entering \[Tournament\].\' CTA: Browse.                  /tournaments/:id

  Prediction won                      Player               P1             In-App + Telegram   \'Prediction correct. +\[X\] tokens.\' CTA: View wallet.                                               /wallet

  Weekly active bonus credited        Player               P3             In-App              \'Weekly active bonus: +\[X\] tokens.\' CTA: View wallet.                                              /wallet

  Re-entry bonus window (post-elim)   Player               P1             In-App + Telegram   \'Re-enter within \[X\] hours for bonus tokens.\' cta_expiry_at = window close.                        /tournaments
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.9 System + Security CTAs**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**                 **Recipient**            **Priority**   **Channel**                 **CRM Action + CTA**                                                                **FE Destination**
  ---------------------------------------- ------------------------ -------------- --------------------------- ----------------------------------------------------------------------------------- ------------------------
  New login detected                       All                      P2             Email                       \'New login from \[device/location\].\' CTA: Review activity.                       /settings?tab=security

  Password reset requested                 All                      P2             Email                       Reset instructions.                                                                 /auth/reset

  Password changed                         All                      P2             Email                       \'Password updated.\' CTA: Review security.                                         /settings?tab=security

  Policy update                            Affected users           P2             Email + In-App              Policy update notice.                                                               /notifications

  Support ticket created                   Ticket owner             P2             Email + In-App              \'Ticket #\[ID\] received.\' CTA: View ticket.                                      /support/tickets/:id

  Support ticket replied                   Ticket owner             P2             Email + In-App              \'New reply on ticket #\[ID\].\' CTA: Reply.                                        /support/tickets/:id

  Service incident affecting tournaments   Affected players + TOs   P0             Email + In-App + Telegram   \'Service issue affecting \[events\].\' CTA: View status page.                      /status

  W-9 threshold approaching (\$550)        Player                   P1             Email + In-App              \'Approaching \$600 tax threshold. Submit W-9 to prevent hold.\' CTA: Submit W-9.   /wallet/w9

  W-9 required (\>\$600 withdraw)          Player                   P0             Email + In-App + Telegram   \'W-9 required to process payout.\' cta_expiry_at = 7 days.                         /wallet/w9
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 10 --- Dashboard POV --- CRM Fields Per Panel**

Each role sees a purpose-built dashboard powered by CRM fields and CTA state. This section maps every dashboard panel to its exact CRM data source and the frontend component that renders it. This is the contract between backend and frontend.

## **10.1 Player Dashboard --- CRM → FE Panel Map**

> *FE: useDashboard({ role: \'player\' }) + useTournamentsMine() + useWallet() + useCtas({ role: \'player\' })*

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dashboard Panel**          **What the Member Sees**                                                                  **CRM Fields**                                                                                               **FE Component**
  ---------------------------- ----------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------ ----------------------------------
  Next Best Action             Single highest-priority CTA. P0 takes over completely. Fallback to P1 content if no P0.   cta_state=open, sorted by cta_priority. cta_expiry_at drives Countdown.                                      \<NextBestActionCard /\>

  Profile Header               Avatar, gamertag, rank badge, XP bar, season arc, streak flame                            player_rank_tier, player_xp, season_xp_current, login_streak_current                                         \<PlayerDashboardHeader /\>

  Token Wallet                 Balance, weekly earn trend, top earner badge, pending drops, cashback                     token_balance, token_lifetime_earned, weekly_token_rank, pending_token_drops                                 \<TokenWalletCard /\>

  Fiat Wallet                  USD balance, winnings retained, W-9 notice if approaching \$600                           wallet_balance_usd, winnings_in_wallet_usd, lifetime_winnings_usd                                            \<FiatWalletCard /\>

  My Tournaments               Cards per tournament: status badge, countdown, check-in/lobby button                      current_tournament_ids with tournament status + checkin_status + cta_state per tournament                    \<MyTournamentsPanel /\>

  Free Entries (Gold only)     \'{N} free entries remaining\' bar. Resets monthly.                                       free_entries_cap_monthly, free_entries_used_this_month, membership_tier                                      Free entries counter (Gold gate)

  Competitive Stats            Win rate, total entries, wins, avg placement, lifetime winnings, season rank              tournament_win_rate, tournament_entries_total, tournament_wins, lifetime_winnings_usd, season_tier_current   Competitive stats panel

  Badge Showcase               3 pinned badges + \'View collection\' link + rarity count                                 badges_earned, badge_count_by_rarity                                                                         Badge showcase + collection grid

  Season Progress              90-day arc: tier, XP bar, milestone rewards, days remaining, leaderboard rank             season_xp_current, season_tier_current, season_id_current                                                    Season progress panel

  Streak + Challenges          Flame + day count + multiplier, daily/weekly challenge progress                           login_streak_current, streak_multiplier_active, weekly_challenge_progress                                    Streak + challenges panel

  My Team                      Team name, role badge, record, pending invite card if applicable                          team_id_current, team_role, pending_team_invites                                                             My Team mini-panel

  Affiliate Panel              Referral link, funnel visualization, earnings to date, months remaining                   referral_code, affiliate_conversions_count, affiliate_earnings_pending, affiliate_months_remaining           Affiliate panel

  Marketplace Feed             Game gear recs, sponsor token drops, limited drops, winnings conversion CTA               games_primary, active_token_drops, winnings_in_wallet_usd (post-payout)                                      Marketplace feed (Phase 2+)

  Upgrade Banner (Free only)   Gold benefits summary with price. Always visible on gated feature hits.                   membership_tier=free                                                                                         Persistent upgrade banner
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **10.2 Team Captain Dashboard --- CRM → FE Panel Map**

> *FE: All player panels PLUS useTeam(teamId). Captain sees everything a player sees + the ops layer below.*

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dashboard Panel**           **What the Member Sees**                                                          **CRM Fields**                                                                      **FE Component**
  ----------------------------- --------------------------------------------------------------------------------- ----------------------------------------------------------------------------------- -----------------------------
  Next Best Action              P0: check-in risk, roster lock, open disputes. Fallback: pending join requests.   cta_state=open, cta_priority=P0, cta_context_type=team\|tournament                  \<NextBestActionCard /\>

  Team Header                   Team name, tier badge, W/L record, active roster count vs max                     captain_team_name, team_record_wins, team_record_entries, team_roster_size_active   \<TeamHeader /\>

  Pending Join Requests         Approve/Reject queue with player profiles: gamertag, rank, games                  pending_join_requests (player rank + game data from player profile)                 \<JoinRequestList /\>

  Pending Invites Sent          Outbound invites with status: pending/accepted/declined. Cancel/Resend.           pending_invites_sent                                                                Pending invites panel

  Roster Panel                  All members: gamertag, rank badge, role, check-in status per tournament           team_roster_ids, team_tournament_checkin_status, team_role per player               \<RosterPanel /\>

  Roster Lock Countdown         Per-tournament lock clock. Urgent styling when \<2h.                              roster_lock_events with datetime + tournament_id                                    \<RosterLockCountdown /\>

  Check-In Compliance           Per tournament: \'X/Y checked in.\' Color-coded green/yellow/red.                 team_tournament_checkin_status                                                      \<CheckInCompliance /\>

  Active Tournaments            Team tournament cards: name, status, bracket link, next match                     team_tournament_ids_active                                                          Active tournaments panel

  Match Confirm/Dispute Queue   Open result confirmations awaiting captain action. P0 if active disputes.         team_active_dispute_ids, open result confirmations                                  Match confirm/dispute queue
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **10.3 Org Admin Dashboard --- CRM → FE Panel Map**

> *FE: useOrg() + useDashboard({ role: \'org_admin\' })*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dashboard Panel**     **What the Member Sees**                                                      **CRM Fields**                                                      **FE Component**
  ----------------------- ----------------------------------------------------------------------------- ------------------------------------------------------------------- -----------------------------
  Next Best Action        P0: at-risk teams, cap limits reached. P1: approaching caps.                  CTA engine filtered to cta_context_type=team\|system for org role   \<NextBestActionCard /\>

  Org Header              Logo, tier badge, org rank                                                    org_name, org_tier, org_rank_tier, org_leaderboard_rank             Org header

  Team Health Panel       Grid of team cards: status color, check-in %, disputes, roster completeness   team_health_map (full JSON)                                         \<TeamHealthPanel /\>

  Seat + Team Cap Meter   Usage bars with P1/P0 upgrade triggers                                        seat_count_used, seat_count_cap, team_count, team_count_cap         \<OrgCapMeter /\>

  Org Wallet + Budget     Total balance + per-team allocations                                          org_wallet_balance_usd, team_budget_allocations                     Org wallet panel

  Token Pool + Stake      Org token balance + active stake display                                      org_token_balance, org_token_stake_active                           Token pool panel

  Affiliate by Team       Per-team referral performance                                                 affiliate_by_team                                                   Affiliate by team analytics

  Org XP + Rank           XP total + tier badge                                                         org_xp, org_rank_tier, org_leaderboard_rank                         Org XP tracker

  GMV Lifetime            Total org marketplace GMV                                                     marketplace_gmv_org_lifetime                                        GMV StatTile

  Sponsor Deal History    Active + historical sponsor agreements                                        sponsor_deal_ids (with deal detail)                                 Sponsor deal history panel

  Recruit Page Preview    Public-facing recruit page preview + copy link                                public_recruit_page_enabled, org_branding_config                    Recruit page preview
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **10.4 Tournament Organizer Dashboard --- CRM → FE Panel Map**

> *FE: useToOps() + useDashboard({ role: \'to\' })*

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dashboard Panel**      **What the Member Sees**                                                **CRM Fields**                                                                                                                             **FE Component**
  ------------------------ ----------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------ --------------------------
  Next Best Action         Highest-priority ops item: disputes, payout blockers, check-in risk     ops_queue with cta_priority mapping                                                                                                        \<NextBestActionCard /\>

  Live Ops Queue           4 count tiles: pending check-ins, disputes, waitlist, payout blockers   ops_queue: {pending_checkins, open_disputes, waitlist_count, payout_blockers}                                                              \<LiveOpsQueue /\>

  Event Pipeline           Active events: fill rate bar, status, surge toggle, share               fill_rate_current_event, tournament status, momentum_surge_active                                                                          \<EventPipelineList /\>

  Event Creation Panel     Create event wizard with inline feature locks                           to_membership_tier → gate evaluation                                                                                                       Event creation wizard

  Reputation / KPI Panel   Score 0--100 with breakdown + reliability streak                        reputation_score, fill_rate_avg, dispute_rate_avg, player_satisfaction_score_avg, time_to_settlement_avg_hrs, reliability_streak_current   \<ReputationPanel /\>

  Sponsor Bid Review       Incoming bids for event slots: accept/reject                            sponsor_deal_ids + bid detail (pending bids)                                                                                               Sponsor bid review panel

  Sold-out Badge Counter   Count of 100%-fill events                                               soldout_badge_count                                                                                                                        Sold-out badge StatTile
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **10.5 Sponsor Dashboard --- CRM → FE Panel Map**

> *FE: useSponsorCampaigns() + useDashboard({ role: \'sponsor\' })*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dashboard Panel**            **What the Member Sees**                                                      **CRM Fields**                                                                **FE Component**
  ------------------------------ ----------------------------------------------------------------------------- ----------------------------------------------------------------------------- ------------------------------
  Next Best Action               Asset upload needed, token pool depleted, renewal due, tier upgrade           cta_engine filtered to cta_context_type=marketplace\|membership for sponsor   \<NextBestActionCard /\>

  Campaign KPI Panel             Impressions, CTR, token redemptions, conversions                              campaign_performance_score, campaigns_active_count, token_campaigns_active    \<CampaignKpiPanel /\>

  Creative Asset Checklist       Logo/banner/copy/overlay --- missing items red; P0 if blocking                creative_asset_checklist JSON                                                 \<CreativeAssetChecklist /\>

  Token Manager                  Token pool balance + active campaigns + multiplier + tokens locked            token_pool_balance, token_pool_locked_amount, token_campaigns_active          Token manager panel

  Category Leaderboard           Rank vs competitors. Category exclusivity lock CTA if rank 1.                 category_leaderboard_rank, category_exclusivity_locked, sponsor_tier          Category leaderboard panel

  ROI Analytics                  Per-campaign impressions trend, CTR, redemptions. Phase 3: GMV attribution.   campaign_performance_history, gmv_attributed_usd (Phase 3)                    ROI dashboard

  Premium Reporting (Upsell)     Advanced cohort analysis --- locked behind premium_reporting tier             dashboard_access_tier                                                         LockedFeatureCard → upgrade

  Tier Progression               Current tier, score to next, renewal streak, loyalty benefits                 sponsor_tier, renewal_streak_count, campaign_performance_score                Tier progression panel

  Billing + Invoices             Current period spend, invoice history, next renewal                           total_spend_lifetime_usd, current_period_spend_usd                            Billing panel

  MVP Naming Rights (Platinum)   Edit + preview MVP badge name in events                                       mvp_naming_rights_active, mvp_badge_name_custom                               MVP naming rights panel
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Section 11 --- CRM Segmentation + AEU Admin View**

## **11.1 Core CRM Segments**

These segments power automations, win-back campaigns, upsell timing, and operational monitoring. Applied across all roles. Each segment maps to one or more CTA triggers in Section 9 and one or more admin panels in Section 11.2.

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Segment Name**              **CRM Logic Definition**                                                                           **Drives**
  ----------------------------- -------------------------------------------------------------------------------------------------- ---------------------------------------------------
  New Signups (0--7d)           created_at \> NOW() - 7d                                                                           Onboarding sequence; profile completion CTAs

  Profile Incomplete            profile_completion_pct \< 80 AND created_at \> NOW() - 30d                                         Nudge sequence D+3

  Free → Gold Candidates        membership_tier=free AND tournament_entries_total \>= 2                                            Upsell card on tournament #2 entry

  Gold At-Risk                  membership_tier=gold AND membership_renewal_date \< NOW() + 14d AND at_risk_churn_score \> 0.5     Win-back sequence; renewal push

  High Value Players            tournament_entries_paid \> 20 OR lifetime_winnings_usd \> 500 OR marketplace_gmv_lifetime \> 200   VIP treatment; priority support routing

  Telegram-Unconnected          telegram_chat_id IS NULL                                                                           Bot connect CTA in-app (P3)

  Telegram-Engaged (7d)         telegram_bot_started=true AND last_telegram_activity_at \> NOW() - 7d                              Primary push delivery segment

  Active Affiliates             affiliate_conversions_count \> 0 AND affiliate_earnings_pending \> 0                               Affiliate payout queue; ambassador comms

  Top Referrers                 affiliate_conversions_count \>= 5                                                                  Ambassador segment; priority comms

  Dormant Players (30d)         last_tournament_entry_at \< NOW() - 30d AND membership_tier=gold                                   Win-back sequence; re-entry bonus offer

  Streak Active (7+ days)       login_streak_current \>= 7                                                                         Protect streak --- avoid disruptive notifications

  W-9 Pre-Threshold             lifetime_winnings_usd \>= 550 AND w9_submitted=false                                               W-9 prompt P1 → P0 at \$600 block

  High-Fill TOs                 fill_rate_avg \>= 0.85                                                                             Priority sponsor matching + featured placement

  At-Risk TOs                   fill_rate_avg \< 0.60                                                                              Ops outreach + performance coaching

  Sponsor Renewal Risk          last_campaign_launched_at \< NOW() - 45d AND renewal_streak_count \> 0                             Renewal CTA P2 → escalate

  Category Lock Candidates      sponsor_tier=Silver AND category_leaderboard_rank=1                                                Fast-track Gold push for category exclusivity

  Orgs Near Seat Cap            seat_count_used / seat_count_cap \> 0.80                                                           Upgrade CTA P1; seat bundle offer

  Captains With Open Requests   pending_join_requests.length \> 0 AND unactioned \> 48h                                            Reminder CTA P1
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **11.2 AEU Admin Dashboard --- CRM Data Surfaced**

> *FE: Internal admin hooks → \<AdminHome /\>. Access restricted to aeu_admin role. All panels correspond to execution tracker items in FE Spec v3 Section 14.*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Admin Panel**          **CRM Data Surfaced**                                                                                     **FE Execution Tracker Item**
  ------------------------ --------------------------------------------------------------------------------------------------------- --------------------------------
  Member Health Overview   account_status, membership_tier, last_login_at, at_risk_churn_score, open support tickets, active flags   ☐ Cross-role filter dashboard

  Revenue Overview         subscription MRR, rake generated, marketplace GMV, affiliate payouts owed, token economy net margin       ☐ Revenue overview

  Affiliate Payout Queue   Pending earnings by referrer, conversion history, 12-month cap meter, payout schedule                     ☐ Affiliate payout queue

  Dispute Pipeline         All open disputes: status, tournament, parties, escalation level, TO dispute rate flags                   ☐ Dispute pipeline monitor

  KYC + W-9 Queue          Users approaching \$600, KYC status, payout hold flags, W-9 submission compliance                         ☐ Compliance queue (KYC + W-9)

  Churn Risk Monitor       at_risk_churn_score \> 0.7, days since last entry, win-back eligibility, membership status                ☐ Churn risk monitor

  TO Quality Monitor       fill_rate below threshold, dispute rate flags, satisfaction alerts, reliability streak status             ☐ CTA engine health

  Sponsor Review Queue     Pending approvals, brand safety flags, enterprise inquiry pipeline, Platinum contract status              ☐ Cross-role filter dashboard

  Token Economy Health     Earn rate vs redemption rate, breakage %, total in circulation, sponsor-funded split %                    ☐ Token economy health

  CTA Engine Health        Open P0 CTAs unactioned \> threshold, expired CTAs not completed, channel delivery failures               ☐ CTA engine health

  Ticker Message Manager   CRUD for Message Board Ticker: message, priority, link, publish_at, unpublish_at                          ☐ Ticker message manager

  System Incident Log      Active incidents, affected user count, notification delivery status, resolution timeline                  ☐ Incident broadcast tool
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------

*AEU CRM Backend Architecture v3.0 \| February 2026 \| Aligned to: AEU Frontend Master Spec v3.0 \| Confidential --- Internal Use Only*


---

# PART 3: OCR Stat Tracking Feature Spec (v1)

+--------------------------------------------------------------------------------------+
| **AMATEUR ESPORTS UNION**                                                            |
|                                                                                      |
| **OCR Stat Tracking Feature Spec**                                                   |
|                                                                                      |
| *Frontend (Mini App + Web) + CRM Backend Architecture*                               |
|                                                                                      |
| Version 1.0 \| February 2026 \| Addendum to Master Spec v3.0 + CRM Architecture v3.0 |
+======================================================================================+
+--------------------------------------------------------------------------------------+

  -----------------------------------------------------------------------------------------------------
  Document Type         Feature Addendum --- OCR Stat Tracking
  --------------------- -------------------------------------------------------------------------------
  Parent Docs           AEU_Master_Spec_v3.0 \| AEU_CRM_Architecture_v3.0

  Surfaces              Telegram Mini App (primary) + Web App

  Affected Roles        player, team_captain, org_admin, aeu_admin

  Phase                 Phase 2 --- Core Feature Rollout

  Audience              Design \| Frontend Engineering \| Backend Engineering \| Product \| Executive

  Classification        Confidential --- Internal Use Only
  -----------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 1 --- FEATURE OVERVIEW + DESIGN PRINCIPLES**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **1.1 What This Feature Does**

The OCR Stat Tracker allows players to photograph or upload their in-game end-of-match stat screens directly from the Telegram Mini App (and web app) immediately after completing a verified AEU match. An AI-powered OCR pipeline extracts structured stat data from the screenshot, normalizes it against a game-specific schema, presents a pre-filled review form to the player for correction, and --- upon confirmation --- writes the data permanently to the player\'s Career Stats panel in their CRM profile.

**This feature closes a critical data gap: AEU currently tracks match results (win/loss/dispute) but has no visibility into how players actually performed. Stat tracking enables four compounding platform benefits:**

  -----------------------------------------------------------------------------------------------------------------------------------
  **Benefit**              **Platform Impact**
  ------------------------ ----------------------------------------------------------------------------------------------------------
  Richer Player Profiles   Stat history makes profiles meaningful beyond W/L record --- drives engagement and social sharing

  Retention + Prestige     Career stat milestones feed badge unlocks, XP awards, and Gold member public profile visibility

  Sponsor Value            Aggregate stat data creates audience intelligence for sponsor targeting and campaign optimization

  TO Credibility           Rich match data strengthens AEU\'s position as a serious competitive infrastructure vs. casual platforms
  -----------------------------------------------------------------------------------------------------------------------------------

## **1.2 Core Design Principles**

-   Match-locked: Stat uploads are only permitted after a match result is confirmed in the AEU system. No freestanding uploads.

-   Review-first: No stat data auto-saves without player review. The player always sees extracted data, corrects errors, and explicitly confirms before anything persists.

-   Confidence-gated: Low AI confidence fields are flagged visually. Submissions below a platform threshold are routed to admin review rather than silently accepted.

-   Gold-public / Free-private: Extracted stats are always stored for all players, but public profile visibility is a Gold membership benefit only.

-   Game-aware: Top competitive titles use structured game-specific schemas. All other games fall back to a generic universal schema. Game support expands over time.

-   Additive to existing architecture: This feature slots into existing CRM fields (player profile, match records), CTA engine, and notification system without restructuring v3.0 contracts.

  -----------------------------------------------------------------------
  **SECTION 2 --- GAME SUPPORT + SCHEMA ARCHITECTURE**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **2.1 Launch Tier Coverage**

Game support is organized into two tiers. Tier 1 games have structured, game-specific field schemas with named stat keys. Tier 2 covers all other games using a generic key-value schema that captures whatever stats are visible in the screenshot without game-specific normalization.

  -----------------------------------------------------------------------------------------------------------------------------------------------------
  **Tier**   **Game**                    **Tracked Stats (Structured)**                                                              **Schema ID**
  ---------- --------------------------- ------------------------------------------------------------------------------------------- ------------------
  1          Valorant                    Kills, Deaths, Assists, ACS, ADR, HS%, KAST%, First Bloods, Plants, Defuses, Agent, Map     schema_valorant

  1          League of Legends           Kills, Deaths, Assists, KDA, CS, CS/min, Gold, Damage, Vision Score, Champion, Role, Lane   schema_lol

  1          Fortnite                    Kills, Placement, Damage Dealt, Accuracy, Eliminations, Revives, Match Duration             schema_fortnite

  1          Call of Duty (Warzone/MP)   Kills, Deaths, KD Ratio, Damage, Assists, Score, Accuracy, Mode, Map                        schema_cod

  1          Apex Legends                Kills, Damage, Placement, Revives, Knocks, Survival Time, Legend                            schema_apex

  1          Rocket League               Goals, Assists, Saves, Shots, Score, Demo Count, Boost Used, Car                            schema_rl

  1          CS2                         Kills, Deaths, Assists, ADR, HS%, KAST%, Rating 2.0, Impact, Map, Side                      schema_cs2

  1          Overwatch 2                 Elims, Deaths, Damage, Healing, Mitigation, Hero, Role, Map, Mode                           schema_ow2

  2          All Other Games             Generic key-value pairs: stat_label + stat_value extracted from visible scoreboard text     schema_generic
  -----------------------------------------------------------------------------------------------------------------------------------------------------

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Expansion Rule**                                                                                                                                                                         |
|                                                                                                                                                                                            |
| New Tier 1 schemas are added based on game volume in AEU tournaments. When a Tier 2 game exceeds 500 cumulative matches, a structured schema is scoped and added in the next sprint cycle. |
+============================================================================================================================================================================================+
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## **2.2 Schema Contract --- TypeScript**

All schemas conform to the following base contract. Game-specific schemas extend StatSchema with typed field definitions.

+-----------------------------------------------------------------------+
| type StatField = {                                                    |
|                                                                       |
| key: string; // e.g. \'kills\', \'adr\', \'placement\'                |
|                                                                       |
| label: string; // Display label in review UI                          |
|                                                                       |
| type: \'integer\' \| \'decimal\' \| \'percentage\' \| \'string\';     |
|                                                                       |
| required: boolean; // Required fields block submission if missing     |
|                                                                       |
| min?: number; // Validation bounds                                    |
|                                                                       |
| max?: number;                                                         |
|                                                                       |
| };                                                                    |
|                                                                       |
| type StatSchema = {                                                   |
|                                                                       |
| schema_id: string; // e.g. \'schema_valorant\'                        |
|                                                                       |
| game_id: string; // Matches games_primary\[\] in player profile       |
|                                                                       |
| version: string; // Schema version --- for future migrations          |
|                                                                       |
| fields: StatField\[\]; // Ordered array of stat field definitions     |
|                                                                       |
| };                                                                    |
|                                                                       |
| type ExtractedStat = {                                                |
|                                                                       |
| key: string;                                                          |
|                                                                       |
| raw_value: string; // Raw OCR output before normalization             |
|                                                                       |
| normalized_value: string \| number; // After type coercion            |
|                                                                       |
| confidence: number; // 0.0--1.0 AI confidence score                   |
|                                                                       |
| flagged: boolean; // true if confidence \< CONFIDENCE_THRESHOLD       |
|                                                                       |
| player_corrected: boolean; // true if player edited this field        |
|                                                                       |
| };                                                                    |
+=======================================================================+
+-----------------------------------------------------------------------+

  -----------------------------------------------------------------------
  **SECTION 3 --- OCR PIPELINE ARCHITECTURE (BACKEND)**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **3.1 Pipeline Overview**

The OCR pipeline is a sequential backend service triggered by the player\'s screenshot upload. It runs asynchronously --- the frontend shows a processing state while the pipeline completes, then delivers results to the review form via React Query refetch or WebSocket event.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Step**   **Service**          **What Happens**
  ---------- -------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1          Upload Gate          Verify: player is authenticated, match_id is valid, match result status = confirmed, upload within submission_window_hours. Reject if any gate fails --- return error code.

  2          Image Intake         Accept image (JPEG/PNG/HEIC/WEBP). Validate file size \<= 15MB. Store raw image to object storage with match_id + player_id namespace. Generate intake_id.

  3          Pre-processing       Resize to optimal OCR resolution. Auto-rotate if needed. Enhance contrast for dark UI game screens. Convert HEIC to JPEG if needed.

  4          Game Detection       AI classifies the game from UI elements, color palette, HUD layout, and font patterns. Returns: game_id + detection_confidence. Falls back to schema_generic if confidence \< 0.75.

  5          OCR Extraction       Run OCR (cloud vision API) on preprocessed image. Extract all text regions with bounding box coordinates.

  6          Schema Mapping       Match extracted text regions to schema field positions using game-specific layout templates. For schema_generic: identify label-value pairs heuristically.

  7          Normalization        Coerce raw strings to typed values per schema (integer, decimal, percentage, string). Handle localized formats (commas as decimal separators, etc.).

  8          Confidence Scoring   Per-field confidence: 0.0--1.0. Fields below CONFIDENCE_THRESHOLD (0.70) are flagged. Overall submission_confidence = mean of all required field scores.

  9          Fraud Pre-check      Validate extracted values against statistical plausibility bounds for the game and mode. Flag outliers (e.g. 80 kills in a 5v5 match). Log flag reason.

  10         Result Delivery      Return ExtractedStatSubmission object to frontend. Player review flow begins. Pipeline complete.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.2 Confidence Threshold Rules**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Condition**                              **System Action**                                   **Player Sees**
  ------------------------------------------ --------------------------------------------------- --------------------------------------------------------------------------------------------------------------
  Field confidence \>= 0.90                  Auto-accept field value                             Field shown with green check indicator. Editable.

  Field confidence 0.70--0.89                Accept with flag                                    Field shown with yellow warning. Prompted to verify.

  Field confidence \< 0.70                   Flag for correction                                 Field highlighted red. Value shown but player must confirm or correct before submitting.

  Submission confidence \>= 0.80 (overall)   Allow direct submission after player review         Normal confirm + save flow.

  Submission confidence \< 0.80 (overall)    Route to admin review queue after player confirms   Player submits normally but sees: \'Submitted --- under review.\' Stats marked pending until admin approves.

  Fraud flag triggered                       Force admin review regardless of confidence         Player sees no fraud indication --- sees standard \'under review\' messaging.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **3.3 Submission Window**

Stat screenshots must be submitted within a defined window after match result confirmation. This prevents historical fabrication and ensures screenshots reflect the actual match played.

-   Default submission window: [2 hours]{.mark} after match result is confirmed in AEU system

-   Window is tied to the specific match_id --- not the tournament end date

-   Window closes automatically --- no manual extension available to players

-   TOs and aeu_admin can extend the window on a per-match basis via admin tools (rare edge case)

-   After window close, the upload button is replaced with a locked state: \'Submission window closed\'

+-----------------------------------------------------------------------------------------------------------------------+
| **CRM Field**                                                                                                         |
|                                                                                                                       |
| submission_window_hours: Integer --- configurable per platform (default 24). Stored in platform config, not per-user. |
+=======================================================================================================================+
+-----------------------------------------------------------------------------------------------------------------------+

  -----------------------------------------------------------------------
  **SECTION 4 --- NEW CRM FIELDS (BACKEND DATA MODEL)**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **4.1 Player Profile --- New Stat Fields**

*Added to Section 3 of AEU_CRM_Architecture_v3.0 --- Player Profile Extended Fields. All fields consumed by useDashboard({ role: \'player\' }) and the new usePlayerStats(player_id) hook.*

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**              **Type**      **Source**     **Notes / Logic**
  --------------------------- ------------- -------------- --------------------------------------------------------------------------------------------------------------------------------------
  stat_submissions\[\]        JSON Array    OCR Pipeline   Array of StatSubmission records. One per approved upload. See 4.2 for StatSubmission schema.

  career_stats_by_game        JSON Object   Computed       {game_id: {field_key: aggregate_value}}. Computed from approved stat_submissions\[\]. Recomputed on each new approval.

  stat_submissions_total      Integer       Computed       Count of all approved submissions. Badge + milestone input.

  stat_submissions_pending    Integer       OCR Pipeline   Submissions currently in admin review queue.

  stat_uploads_today          Integer       Rate limiter   Resets at UTC midnight. Gate: max 20 uploads per day per player. Prevents bulk fabrication.

  stats_public_visibility     Boolean       Derived        True if membership_tier = gold. False = stats stored but not shown on public profile. Never manually settable --- derived from tier.

  ocr_xp_earned_total         Integer       XP Engine      Total XP earned from stat submission events. Feeds player_xp.

  stat_milestone_badges\[\]   JSON Array    Badge Engine   Badges unlocked by stat milestone events. E.g. \'First Upload\', \'100 Kills Tracked\', \'Perfect KDA\'.

  ocr_fraud_flags_lifetime    Integer       Admin          Cumulative fraud flags received. Internal only --- never surfaced to player. Admin risk scoring input.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **4.2 StatSubmission Record Schema**

Each upload generates one StatSubmission record. This is the source record for all aggregation, admin review, and audit trail.

+-----------------------------------------------------------------------------------------------+
| type StatSubmission = {                                                                       |
|                                                                                               |
| submission_id: UUID; // Unique submission record ID                                           |
|                                                                                               |
| player_id: UUID; // Links to aeu_contact_id                                                   |
|                                                                                               |
| match_id: UUID; // Verified AEU match --- confirmed result required                           |
|                                                                                               |
| tournament_id: UUID; // Parent tournament of the match                                        |
|                                                                                               |
| game_id: string; // Detected or player-corrected game                                         |
|                                                                                               |
| schema_id: string; // Schema used for extraction                                              |
|                                                                                               |
| image_url: string; // Object storage URL of original screenshot                               |
|                                                                                               |
| extracted_stats: ExtractedStat\[\]; // Raw pipeline output --- per-field                      |
|                                                                                               |
| player_confirmed_stats: Record\<string, string\|number\>; // Final values after player review |
|                                                                                               |
| submission_confidence: number; // Mean confidence across required fields (0.0--1.0)           |
|                                                                                               |
| fraud_flagged: boolean; // Internal flag --- never surfaced to player                         |
|                                                                                               |
| fraud_flag_reason?: string; // Admin-visible reason string                                    |
|                                                                                               |
| review_status: \'pending\' \| \'approved\' \| \'rejected\' \| \'auto_approved\';              |
|                                                                                               |
| reviewed_by?: UUID; // Admin user_id if manually reviewed                                     |
|                                                                                               |
| reviewed_at?: Timestamp;                                                                      |
|                                                                                               |
| rejection_reason?: string; // Shown to player on rejection                                    |
|                                                                                               |
| submitted_at: Timestamp; // Player confirmation time                                          |
|                                                                                               |
| approved_at?: Timestamp; // When added to career_stats_by_game aggregate                      |
|                                                                                               |
| };                                                                                            |
+===============================================================================================+
+-----------------------------------------------------------------------------------------------+

## **4.3 Aggregation Logic --- career_stats_by_game**

career_stats_by_game is a computed field. It is never written directly --- it is always recomputed from approved stat_submissions\[\]. This ensures that admin rejection of a previously approved submission correctly removes its contribution.

  -------------------------------------------------------------------------------------------------------------------------
  **Stat Type**               **Aggregation Method**                          **Example**
  --------------------------- ----------------------------------------------- ---------------------------------------------
  Count stats                 Sum across all approved submissions for game    Kills: 847 total across 62 Valorant matches

  Rate stats                  Weighted average (weighted by matches played)   HS%: 28.4% avg across all CS2 submissions

  Performance scores          Mean average                                    ACS: 214.3 avg across Valorant submissions

  Best single-match records   Max value tracked separately as stat_peak\[\]   Best KDA: 18/2/11 in League of Legends

  Placement stats             Average and best tracked separately             Avg Placement: 4.2 \| Best: #1 (Fortnite)
  -------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 5 --- FRONTEND SPEC (MINI APP + WEB)**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **5.1 New Frontend Hooks**

*Added to Section 16 of AEU_Master_Spec_v3.0 --- Required Hook List.*

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Hook**                    **Returns**                                                                                         **Notes**
  --------------------------- --------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------
  usePlayerStats(player_id)   career_stats_by_game, stat_submissions_total, stat_submissions_pending, stat_milestone_badges\[\]   Powers Career Stats panel on player profile. Public for Gold; private for Free (own profile only).

  useStatUpload(match_id)     upload state, extraction result, review form state, submission state                                Powers the full OCR upload flow. Scoped to one match. Manages all pipeline states.

  useStatSchema(game_id)      StatSchema for the given game                                                                       Fetched once per session per game. Cached. Used to render review form fields in correct order.

  useAdminStatQueue()         Pending StatSubmission\[\] for review                                                               Admin only. Powers stat review queue in /admin dashboard.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.2 New Required Mutation**

*Added to Section 16.2 of AEU_Master_Spec_v3.0 --- Required Mutations.*

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Mutation**                                            **Optimistic?**   **On Success**                                                 **On Error**
  ------------------------------------------------------- ----------------- -------------------------------------------------------------- --------------------------------------------
  submitStatUpload(match_id, image)                       No                Trigger pipeline → return to review form with extracted data   Error state in upload panel + retry button

  confirmStatSubmission(submission_id, confirmed_stats)   No                Refetch usePlayerStats + success card + XP award CTA           Error toast + keep review form open

  adminApproveStatSubmission(submission_id)               Yes               Refetch useAdminStatQueue + aggregate recomputed               Revert + toast

  adminRejectStatSubmission(submission_id, reason)        Yes               Refetch useAdminStatQueue + player notified via CTA            Revert + toast
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.3 OCR Upload Flow --- 7-Step UI State Map**

*Added to Section 5.5 of AEU_Master_Spec_v3.0 --- Player Workflow. Occurs post-match after result is confirmed.*

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Step**   **Screen / Component**                     **UI State**                                                                                                                                                                    **Priority**
  ---------- ------------------------------------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------------------
  1          Post-match result screen                   P1 CTA card appears: \'Upload your stat screenshot to track your performance.\' Upload button + \'Skip\' option.                                                                P1

  2          Upload panel (/match/:matchId?tab=stats)   Camera/gallery picker. Shows match context header: game, opponent, result. File size warning displayed. Accepts JPEG, PNG, HEIC, WEBP.                                          P1

  3          Processing state                           Full-screen loading card. Skeleton placeholder replacing upload button. Text: \'Reading your stats\...\' spinner animation. On Telegram: BottomSheet with progress indicator.   P1

  4          Review form --- high confidence            Pre-filled form with all extracted fields. Green check on fields \>= 0.90 confidence. Yellow caution icon on fields 0.70--0.89. Game detected label shown.                      P0

  5          Review form --- low confidence fields      Red border on fields \< 0.70. Cannot submit until red fields are manually confirmed or corrected. \'Needs review\' badge count shown at top of form.                            P0

  6          Confirm & Submit                           Primary button: \'Save Stats\'. Tapping shows brief summary card of what will be saved. Player taps confirm to finalize.                                                        P1

  7          Success / Pending state                    If auto-approved: success card + XP earned + \'View your career stats\' CTA. If pending admin review: \'Stats submitted --- under review\' card. No XP until approved.          P1
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **5.4 Review Form Component Spec**

+----------------------------------------------------------------------------------+
| type StatReviewFormProps = {                                                     |
|                                                                                  |
| submission_id: string;                                                           |
|                                                                                  |
| game_id: string; // Used to label game + fetch schema                            |
|                                                                                  |
| game_detected: boolean; // True if auto-detected, false if player selected       |
|                                                                                  |
| schema: StatSchema; // Field definitions for render order + validation           |
|                                                                                  |
| extracted: ExtractedStat\[\]; // Pre-fills each field; confidence drives styling |
|                                                                                  |
| onConfirm: (confirmed: Record\<string, string\|number\>) =\> void;               |
|                                                                                  |
| onRetake: () =\> void; // Triggers new upload --- discards current extraction    |
|                                                                                  |
| };                                                                               |
+==================================================================================+
+----------------------------------------------------------------------------------+

**Review Form Rendering Rules**

-   Fields rendered in schema-defined order --- never alphabetically

-   Each field shows: label, current value (editable input), confidence indicator icon (green/yellow/red)

-   Red fields must be interacted with (tapped/clicked) before submit button activates --- prevents accidental confirmation of wrong data

-   \'Wrong game?\' link at top of form --- opens game selector dropdown. Resubmits image with selected game_id to force schema re-detection. Does not re-run OCR --- re-runs schema mapping only.

-   \'Retake screenshot\' option available --- discards current extraction and returns to upload step

-   On Telegram: form renders as stacked card fields in BottomSheet. No multi-column layout.

-   On Web: two-column layout for Tier 1 games (stat fields side by side). Single column for schema_generic.

## **5.5 Career Stats Panel --- Player Profile**

*Added to Section 5 of AEU_Master_Spec_v3.0 --- Player Dashboard, Membership Features & Workflow.*

The Career Stats panel displays aggregated stat history on the player\'s profile. It is powered by career_stats_by_game from the CRM and rendered by usePlayerStats(player_id).

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Panel State**                       **What Renders**
  ------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------
  No submissions yet                    EmptyState: \'No stats tracked yet. Upload your first screenshot after your next match.\' CTA to /match/:matchId if active match exists.

  Has submissions, Free member          Stats visible to the player in their own profile. Public profile shows locked panel: \'Upgrade to Gold to display your career stats publicly.\' LockedFeatureCard.

  Has submissions, Gold member          Full stats panel visible on public profile. Game tabs across top. Per-game stat grid with aggregate values + peak records. \'X matches tracked\' count.

  Pending submissions                   Pending badge: \'X submissions under review.\' Stats shown but pending values shown with clock icon instead of value until approved.

  Submission window open (post-match)   Upload prompt card visible inside Career Stats panel: \'Upload your \[game\] stats from your match vs \[opponent\].\' Upload CTA.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Panel Layout --- Game Tabs**

-   Each game the player has submitted stats for appears as a tab. Tab shows game name + number of matches tracked.

-   Active tab shows: aggregate stat grid (StatTile components), peak record highlight cards, match count + date range

-   On Telegram: game tabs replaced by a vertical accordion list --- one section per game, collapsible

-   Stats with zero submissions for a game are never shown --- empty games are not rendered as tabs

## **5.6 Accessibility + Platform Constraints**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Requirement**         **Web App**                                              **Telegram Mini App**
  ----------------------- -------------------------------------------------------- -------------------------------------------------------------------------------
  Upload trigger          File input + drag-drop zone                              Native Telegram camera/gallery picker via WebApp API --- no custom file input

  Processing state        Inline skeleton in panel                                 BottomSheet with pulsing indicator --- no full-screen takeover

  Review form layout      Two-column for Tier 1 games; single column for generic   Single column stacked fields only --- no tables

  Confidence indicators   Color + icon + text label                                Color + icon only --- no inline text (space constraint)

  Confirm step            Modal overlay with summary                               BottomSheet confirmation --- not full-screen modal

  Career stats display    Full grid with charts (future)                           StatTile stack per game --- no charts, no tables

  Keyboard nav            Full keyboard operability; focus ring on all inputs      Not applicable (touch only)

  Aria labels             All confidence icons + stat inputs require aria-label    Not required for Telegram surface
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 6 --- CTA TRIGGERS + NOTIFICATION ROUTING**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **6.1 New CTA Trigger Rules**

*Added to Section 9.3 and 9.8 of AEU_CRM_Architecture_v3.0 --- CTA Trigger Map.*

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger / Microevent**                          **Priority**   **Channel**         **CTA Text**                                                                                      **FE Destination**
  ------------------------------------------------- -------------- ------------------- ------------------------------------------------------------------------------------------------- ---------------------------
  Match result confirmed + submission window open   P1             In-App + Telegram   \'Upload your stat screenshot --- window open for 24h.\' CTA: Upload now.                         /match/:matchId?tab=stats

  Submission window closing in 2h                   P1             In-App + Telegram   \'Stat upload closes in 2 hours for \[match\].\' CTA: Upload now. cta_expiry_at = window close.   /match/:matchId?tab=stats

  Submission window closed --- no upload made       P3             In-App              \'Window closed. Upload stats after your next match.\' No action CTA --- informational only.      /home

  Stat submission auto-approved                     P1             In-App + Telegram   \'Stats saved! +\[X\] XP earned.\' CTA: View career stats.                                        /profile?tab=stats

  Stat submission approved by admin                 P1             In-App + Telegram   \'Your stats from \[match\] have been approved.\' CTA: View career stats.                         /profile?tab=stats

  Stat submission rejected by admin                 P2             In-App + Email      \'Your stat submission was not approved: \[reason\].\' CTA: View details.                         /match/:matchId?tab=stats

  Stat milestone reached (career)                   P1             In-App + Telegram   \'Career milestone: \[milestone name\] --- badge unlocked.\' CTA: View badge.                     /profile?tab=badges

  Fraud flag triggered (internal)                   P0             Admin In-App only   Admin sees: \'Flagged submission --- \[player\] --- \[match\].\' CTA: Review.                     /admin?tab=stat_review
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 7 --- ADMIN PANEL ADDITIONS**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **7.1 Admin Stat Review Queue**

*Added to Section 14 of AEU_Master_Spec_v3.0 --- Admin Dashboard Execution Tracker.*

The Stat Review Queue is a new panel in the admin dashboard. It surfaces all StatSubmission records with review_status = \'pending\'. Access restricted to aeu_admin role.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Panel Feature**                      **Implementation Notes**
  -------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Pending queue list                     List of pending submissions sorted by: fraud_flagged first, then submission_confidence ascending (lowest confidence at top), then submitted_at oldest first.

  Submission detail view                 Shows: screenshot image, extracted_stats vs player_confirmed_stats side by side, confidence per field, game detected, match context, player profile link, fraud flag reason if applicable.

  Approve action                         Sets review_status = \'approved\'. Triggers career_stats_by_game recomputation. Fires approval CTA to player. Awards XP.

  Reject action                          Requires rejection_reason input (dropdown + optional custom text). Sets review_status = \'rejected\'. Fires rejection notification to player. Increments ocr_fraud_flags_lifetime if fraud_flagged.

  Bulk approve (high confidence batch)   Approve all pending submissions with submission_confidence \>= 0.80 AND fraud_flagged = false in one action. Confirmation dialog required.

  Queue health StatTiles                 Count tiles: Total Pending \| Fraud Flagged \| Avg Confidence \| Oldest Submission Age (hours).

  Player risk indicator                  If ocr_fraud_flags_lifetime \>= 3: show risk badge on player name in queue. Admin can view player\'s full submission history.
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 8 --- GAMIFICATION + MEMBERSHIP INTEGRATION**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **8.1 XP Awards --- Stat Upload Events**

*Added to Section 5.6 of AEU_Master_Spec_v3.0 --- Gamification & Social --- Player.*

  --------------------------------------------------------------------------------------------------------------------------------------
  **Event**                                    **XP Awarded**   **Notes**
  -------------------------------------------- ---------------- ------------------------------------------------------------------------
  First stat submission ever (any game)        +100 XP          One-time only. \'First Upload\' badge also unlocked.

  Each approved stat submission                +25 XP           Per approved submission. Not awarded for pending --- only on approval.

  Tier 1 game submission (structured schema)   +10 XP bonus     Added on top of base +25 XP for using a supported game schema.

  10th approved submission (any game)          +50 XP           Milestone bonus. Badge: \'Stat Tracker\'.

  50th approved submission                     +150 XP          Milestone bonus. Badge: \'Performance Analyst\'.

  100th approved submission                    +500 XP          Milestone bonus. Badge: \'Stat Legend\' (Legendary rarity).
  --------------------------------------------------------------------------------------------------------------------------------------

## **8.2 Stat Milestone Badges**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Badge Name**              **Trigger Condition**                                       **Rarity**   **Notes**
  --------------------------- ----------------------------------------------------------- ------------ -----------------------------------------------------------------------------------------------
  First Upload                First approved stat submission                              Common       Awarded to all players on first upload.

  Stat Tracker                10 approved submissions                                     Rare         Cross-game --- any combination of games.

  Performance Analyst         50 approved submissions                                     Epic         Cross-game milestone.

  Stat Legend                 100 approved submissions                                    Legendary    Top tier badge --- high prestige display value.

  Game Specialist: \[Game\]   25 approved submissions for one specific game               Rare         One badge per game. E.g. \'Game Specialist: Valorant\'.

  Peak Performer              Single-match stat in top 1% for that game (platform-wide)   Epic         Requires sufficient data volume --- activated after 1,000 approved submissions for that game.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **8.3 Membership Gate --- Public Stats Visibility**

Career stats are always stored for all players regardless of membership tier. The membership gate controls public profile visibility only.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Membership**   **Career Stats Stored?**   **Public Profile Visibility**
  ---------------- -------------------------- ---------------------------------------------------------------------------------------------------------------------------------------
  Free             Yes --- always             Hidden on public profile. LockedFeatureCard shown: \'Career Stats --- Gold members display full stat history publicly.\' Upgrade CTA.

  Gold             Yes                        Fully visible on public profile. Game tabs, aggregate stats, peak records, submission count all displayed.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Revenue Logic**                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                    |
| The public stats panel is a meaningful Gold retention driver. Players who build stat history have a compounding reason to maintain Gold membership --- their profile becomes more valuable over time. Loss of Gold = stats go private = strong retention pressure. |
+====================================================================================================================================================================================================================================================================+
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

  -----------------------------------------------------------------------
  **SECTION 9 --- EXECUTION TRACKER**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## **9.1 Backend Pipeline --- Execution Tracker**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------
       **Feature**                            **Implementation Notes**
  ---- -------------------------------------- ------------------------------------------------------------------------------------------------------------------------
  ☐    Upload gate service                    Validate: auth, match_id exists, result confirmed, within submission_window_hours, stat_uploads_today \< 20

  ☐    Image intake + storage                 Accept JPEG/PNG/HEIC/WEBP. Max 15MB. Store to object storage with match_id/player_id namespace. Return intake_id.

  ☐    Image pre-processing service           Resize, auto-rotate, contrast enhancement, HEIC conversion.

  ☐    AI game detection                      Classify game from UI elements. Confidence threshold 0.75 for Tier 1. Fall back to schema_generic below threshold.

  ☐    OCR extraction service                 Cloud Vision API integration. Return text regions with bounding boxes.

  ☐    Schema mapping engine                  Game-specific layout templates for Tier 1. Heuristic label-value pairing for schema_generic.

  ☐    Value normalization                    Type coercion per StatField.type. Handle locale variants (comma decimals, percent formatting).

  ☐    Confidence scoring                     Per-field 0.0--1.0. Overall submission_confidence = mean of required fields.

  ☐    Fraud plausibility check               Statistical bounds per game + mode. Flag outliers. Log fraud_flag_reason.

  ☐    StatSubmission record creation         Write record with review_status = \'pending\' or \'auto_approved\' per confidence rules.

  ☐    career_stats_by_game aggregation       Recompute on every approval/rejection. Never written directly.

  ☐    Admin review queue API                 Endpoints: GET /admin/stat-review (paginated), POST /admin/stat-review/:id/approve, POST /admin/stat-review/:id/reject

  ☐    Submission window enforcement (cron)   Auto-expire open upload CTAs and flag matches past submission_window_hours.

  ☐    Rate limiting                          stat_uploads_today gate --- reset at UTC midnight via cron.

  ☐    Game schema registry                   Versioned schema definitions stored in config. API endpoint: GET /schemas/:game_id

  ☐    XP award on approval                   Fire XP event to gamification engine on review_status change to approved.

  ☐    Badge milestone checks                 Evaluate stat_milestone_badges\[\] on each approval. Fire badge unlock event if threshold met.
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **9.2 Frontend --- Execution Tracker**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       **Feature**                                            **Implementation Notes**
  ---- ------------------------------------------------------ -------------------------------------------------------------------------------------------------------------------------------------------------------------
  ☐    Post-match upload CTA (P1 card)                        Renders on post-match result screen when submission window is open. Includes match context + 24h countdown.

  ☐    Upload panel (/match/:matchId?tab=stats)               File picker (web) / Telegram media picker. Shows match header. File type + size validation client-side before upload.

  ☐    Processing state (Skeleton + spinner)                  Full skeleton replaces upload zone. Text: \'Reading your stats\...\' On Telegram: BottomSheet with pulsing indicator.

  ☐    StatReviewForm component                               Pre-filled inputs. Confidence indicators per field (green/yellow/red). \'Wrong game?\' selector. \'Retake\' option. Submit gated until red fields resolved.

  ☐    Confidence indicator icons                             Green checkmark \>= 0.90. Yellow caution 0.70--0.89. Red flag \< 0.70. All have aria-labels on web.

  ☐    Confirm summary modal (web) / BottomSheet (Telegram)   Shows confirmed stat values before final save. Requires explicit tap/click to commit.

  ☐    Success card (auto-approved)                           XP earned animation + badge unlock if applicable. CTA: View career stats.

  ☐    Pending review card                                    \'Stats submitted --- under review.\' No XP animation. Clock icon. CTA: View profile.

  ☐    Career Stats panel (player profile)                    Game tabs. Aggregate stat grid (StatTile). Peak record highlight cards. Submission count + date range. Empty/locked/pending states all implemented.

  ☐    Free member gate (public profile)                      LockedFeatureCard on Career Stats section of public profile. Feature name, 3 benefits bullets, Gold upgrade CTA.

  ☐    Submission window closed state                         \'Submission window closed\' locked state replaces upload button after window expires.

  ☐    Telegram constraints (all steps)                       BottomSheet modals, single-column review form, no tables, StatTile grid (no charts), no multi-column layout.

  ☐    Admin stat review panel (/admin)                       Queue list + detail view + approve/reject actions + bulk approve + queue health StatTiles + player risk badge.

  ☐    usePlayerStats hook                                    Returns career_stats_by_game, stat_submissions_total, stat_submissions_pending, stat_milestone_badges\[\]

  ☐    useStatUpload hook                                     Manages full upload state machine: idle → uploading → processing → reviewing → confirming → success/pending

  ☐    useStatSchema hook                                     Fetches + caches game schema. Used to render review form in correct field order.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

+------------------------------------------------------------------------------------------------------------------+
| AEU OCR Stat Tracking Feature Spec v1.0 \| February 2026 \| Addendum to Master Spec v3.0 + CRM Architecture v3.0 |
|                                                                                                                  |
| Confidential --- Internal Use Only --- Not for Distribution                                                      |
+==================================================================================================================+
+------------------------------------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------+
| **AMATEUR ESPORTS UNION**                                                      |
|                                                                                |
| **OCR Stat Tracking --- Sports Games Addendum**                                |
|                                                                                |
| **Madden NFL \| EA Sports CFB \| EA FC \| NBA 2K \| NHL \| MLB The Show**      |
|                                                                                |
| *Team Stats Only --- Frontend (Mini App + Web) + CRM Backend Schema Extension* |
|                                                                                |
| Version 2.0 \| February 2026 \| Addendum to OCR Stat Tracking Spec v1.0        |
+================================================================================+
+--------------------------------------------------------------------------------+

  -----------------------------------------------------------------------------------------------------------
  Document Type         Feature Addendum --- Sports Games OCR Stat Tracking v2.0
  --------------------- -------------------------------------------------------------------------------------
  Parent Docs           AEU_OCR_StatTracking_Spec_v1.0 \| AEU_Master_Spec_v3.0 \| AEU_CRM_Architecture_v3.0

  Stat Capture Scope    Team stats only --- no individual player or position-based tracking

  Titles In Scope       Madden NFL, EA Sports CFB, EA FC, NBA 2K, NHL, MLB The Show

  Modes In Scope        Head-to-Head (H2H), Ultimate Team ranked modes, NBA 2K Pro-Am Online + Play Now

  Submission Window     24 hours post confirmed match result --- same as esports titles

  Gamification          Shared badge + XP system with esports OCR spec --- no separate track

  Classification        Confidential --- Internal Use Only
  -----------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 1 --- SCOPE + DESIGN DECISIONS**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## 1.1 Team Stats Only --- Design Rationale

All six sports titles extract team-level stats from the post-game summary screen. This is the box score view showing the user\'s team aggregate performance for the match --- not individual player cards or position-specific breakdowns. This approach removes position detection complexity entirely, produces consistent extraction across all modes, and delivers immediately meaningful career aggregates.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Decision**                               **Implication**
  ------------------------------------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Team stats only                            No position detection step in pipeline. Schema mapping targets the user\'s team row on the post-game box score screen only.

  No individual player tracking              No per-player card extraction. No position-scoped field sets. One flat schema per game per mode.

  User team row identification               Pipeline identifies the user\'s team row via gamertag match, highlight color, or Home/Away indicator depending on the game\'s screen layout.

  Simpler aggregation                        career_stats hierarchy is game -\> mode only. No position third level. Per-game averages, season totals, H2H record, and peak values all computed at team level.

  Madden/CFB disambiguation still required   These two games share near-identical post-game screen layouts regardless of stat scope. A dedicated disambiguation step remains in the pipeline.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 1.2 In-Scope Modes Per Title

  -------------------------------------------------------------------------------------------------------------------------------------------------
  **Title**       **In-Scope Modes**                                                  **Out-of-Scope Modes**
  --------------- ------------------------------------------------------------------- -------------------------------------------------------------
  Madden NFL      Head-to-Head (H2H), Ultimate Team (MUT) Ranked                      Franchise, Face of Franchise, Superstar Mode, offline vs AI

  EA Sports CFB   Head-to-Head (H2H), Ultimate Team (CUT) Ranked                      Dynasty, Road to Glory, offline vs AI

  EA FC           Head-to-Head (H2H), Ultimate Team (FUT) Rivals + Champions          Career Mode, Volta, Pro Clubs, offline vs AI

  NBA 2K          Head-to-Head (H2H), MyTeam Ranked, Pro-Am Online, Play Now Online   MyCareer offline, Rec (non-competitive), offline vs AI

  NHL             Head-to-Head (H2H), Ultimate Team (HUT) Rivals + Champions          Be a Pro, Franchise, offline vs AI

  MLB The Show    Head-to-Head (H2H), Diamond Dynasty (DD) Ranked Seasons + Events    Road to the Show, Franchise, offline vs AI
  -------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 2 --- TEAM STAT SCHEMAS PER GAME**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

All schemas conform to the base StatSchema contract from OCR Stat Tracking Spec v1.0 Section 2.2. Sports schemas extend with mode_id to scope extraction to the correct post-game screen layout. position_aware is always false --- team stats are flat across all positions by design.

+----------------------------------------------------------------------------------------------+
| // Sports schema extension --- team stats only                                               |
|                                                                                              |
| type SportsStatSchema = StatSchema & {                                                       |
|                                                                                              |
| mode_id: string; // e.g. \'h2h\' \| \'mut_ranked\' \| \'fut_rivals\' \| \'dd_ranked\'        |
|                                                                                              |
| mode_label: string; // Display label: \'Head-to-Head\' \| \'Ultimate Team Ranked\'           |
|                                                                                              |
| position_aware: false; // Always false --- no position detection for sports games            |
|                                                                                              |
| team_context: true; // Always true --- all fields are user team aggregates                   |
|                                                                                              |
| };                                                                                           |
|                                                                                              |
| // Extended submission record for sports games                                               |
|                                                                                              |
| type SportStatSubmission = StatSubmission & {                                                |
|                                                                                              |
| mode_id: string; // Detected or player-confirmed mode                                        |
|                                                                                              |
| mode_detected: boolean; // true = auto-detected; false = player selected                     |
|                                                                                              |
| match_result: \'W\' \| \'L\' \| \'OTL\' \| \'SOL\' \| \'D\'; // Game outcome from screenshot |
|                                                                                              |
| score_user: string; // User team final score                                                 |
|                                                                                              |
| score_opponent: string; // Opponent team final score                                         |
|                                                                                              |
| flag_reason_mode?: string; // \'mode_undetected\' \| \'madden_cfb_ambiguous\'                |
|                                                                                              |
| };                                                                                           |
+==============================================================================================+
+----------------------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------+
| **2.1 Madden NFL**                                                                        |
|                                                                                           |
| *schema_madden_h2h \| schema_madden_mut --- Team Stats --- Post-Game Team Summary Screen* |
+===========================================================================================+
+-------------------------------------------------------------------------------------------+

Stats extracted from the user\'s team row on the Madden post-game summary screen. H2H and MUT Ranked share identical field definitions --- mode_id differentiates for display and aggregation grouping only.

  --------------------------------------------------------------------------------------------
  **Team Stat**            **Type**   **Notes**
  ------------------------ ---------- --------------------------------------------------------
  Total Yards              Integer    Net total offensive yards. Required.

  Passing Yards            Integer    Team passing yards total. Required.

  Rushing Yards            Integer    Team rushing yards total. Required.

  Total Touchdowns         Integer    All TDs (passing + rushing + special teams). Required.

  Turnovers                Integer    INTs thrown + fumbles lost. Required.

  Time of Possession       String     MM:SS format. e.g. \'32:14\'. Optional.

  Third Down Conversions   String     X/Y format. e.g. \'7/12\'. Optional.

  Sacks Allowed            Integer    Times user QB was sacked. Optional.

  Penalties                Integer    Total penalty count against user\'s team. Optional.

  Final Score (user)       Integer    Required. Used to compute match_result.

  Final Score (opponent)   Integer    Required. Used to compute match_result.
  --------------------------------------------------------------------------------------------

-   Fraud bounds: Total Yards \> 1,000 = flag. Total TDs \> 15 = flag. Turnovers \> 12 = flag.

-   Match result enum: W \| L \| D only. No OTL or SOL for Madden.

+-------------------------------------------------------------------------------------+
| **2.2 EA Sports College Football (CFB)**                                            |
|                                                                                     |
| *schema_cfb_h2h \| schema_cfb_cut --- Team Stats --- Post-Game Team Summary Screen* |
+=====================================================================================+
+-------------------------------------------------------------------------------------+

CFB uses an identical field set to Madden. The post-game summary screen layout is nearly indistinguishable between the two titles --- game detection is the critical differentiator. CFB allows a slightly higher ceiling on yardage and touchdowns due to the faster pace of the college game.

  ------------------------------------------------------------------------------
  **Team Stat**            **Type**   **Notes**
  ------------------------ ---------- ------------------------------------------
  Total Yards              Integer    Required.

  Passing Yards            Integer    Required.

  Rushing Yards            Integer    Required.

  Total Touchdowns         Integer    Required.

  Turnovers                Integer    INTs + fumbles lost. Required.

  Time of Possession       String     MM:SS. Optional.

  Third Down Conversions   String     X/Y. Optional.

  Sacks Allowed            Integer    Optional.

  Penalties                Integer    Optional.

  Final Score (user)       Integer    Required.

  Final Score (opponent)   Integer    Required.
  ------------------------------------------------------------------------------

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Madden / CFB Disambiguation**                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                                                                                         |
| When game detection confidence \< 0.80 between these two titles, apply a dedicated disambiguation step using college team logos, uniform design, field markings, and the EA CFB watermark. If unresolved, set flag_reason_mode = \'madden_cfb_ambiguous\' and route to admin review. Never auto-assign to either title. |
+=========================================================================================================================================================================================================================================================================================================================+
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

-   Fraud bounds: Total Yards \> 1,100 = flag. Total TDs \> 18 = flag. Turnovers \> 12 = flag.

-   Match result enum: W \| L \| D only.

+---------------------------------------------------------------------------------------+
| **2.3 EA FC (FIFA)**                                                                  |
|                                                                                       |
| *schema_eafc_h2h \| schema_eafc_fut --- Team Stats --- Post-Game Team Summary Screen* |
+=======================================================================================+
+---------------------------------------------------------------------------------------+

EA FC\'s post-game screen shows team-level stats for both sides. The user\'s team row is identified by team badge and name on the results screen. FUT Rivals and FUT Champions share the same field set --- mode_id differentiates for aggregation and display.

  -----------------------------------------------------------------------------------------
  **Team Stat**            **Type**     **Notes**
  ------------------------ ------------ ---------------------------------------------------
  Goals Scored             Integer      User team total goals. Required.

  Goals Conceded           Integer      Opponent goals. Required. Drives match_result.

  Shots                    Integer      Total shots taken by user team. Required.

  Shots on Target          Integer      Shots on target by user team. Required.

  Possession %             Percentage   Ball possession for user team. 0--100. Required.

  Pass Accuracy %          Percentage   Passing accuracy for user team. 0--100. Required.

  Fouls Committed          Integer      Optional.

  Yellow Cards             Integer      Optional.

  Red Cards                Integer      Optional.

  Corners                  Integer      Corner kicks won. Optional.

  Final Score (user)       Integer      Same as Goals Scored. Required.

  Final Score (opponent)   Integer      Same as Goals Conceded. Required.
  -----------------------------------------------------------------------------------------

-   Fraud bounds: Goals Scored \> 20 = flag. Shots \> 40 = flag. Possession % outside 0--100 = auto-flag.

-   Match result enum: W \| L \| D only. No OTL or SOL for EA FC.

-   FUT Rivals and FUT Champions are distinct mode sub-tabs in career stats --- both map to schema_eafc_fut with different mode_label values.

+--------------------------------------------------------------------------------------------+
| **2.4 NBA 2K**                                                                             |
|                                                                                            |
| *schema_2k_h2h \| schema_2k_myteam \| schema_2k_proam \| schema_2k_playnow --- Team Stats* |
+============================================================================================+
+--------------------------------------------------------------------------------------------+

NBA 2K tracks team-level box score stats from the post-game summary screen. The user\'s team row is identified by gamertag match or the highlighted team name. All four in-scope modes share the same field set --- mode_id determines display grouping only.

  -------------------------------------------------------------------------------------------------
  **Team Stat**                **Type**     **Notes**
  ---------------------------- ------------ -------------------------------------------------------
  Points                       Integer      User team total points. Required.

  Opponent Points              Integer      Opponent total. Required. Drives match_result.

  Field Goals Made/Attempted   String       X/Y format. e.g. \'38/82\'. Required.

  Field Goal %                 Percentage   0--100. Extracted or computed from FGM/FGA. Required.

  3-Pointers Made/Attempted    String       X/Y format. Required.

  3-Point %                    Percentage   0--100. Required.

  Free Throws Made/Attempted   String       X/Y format. Optional.

  Free Throw %                 Percentage   0--100. Optional.

  Total Rebounds               Integer      Offensive + defensive combined. Required.

  Assists                      Integer      Team total. Required.

  Turnovers                    Integer      Team total. Required.

  Steals                       Integer      Team total. Optional.

  Blocks                       Integer      Team total. Optional.

  Final Score (user)           Integer      Same as Points. Required.

  Final Score (opponent)       Integer      Required.
  -------------------------------------------------------------------------------------------------

-   Fraud bounds: Points \> 200 = flag. FG%/3PT%/FT% outside 0--100 = auto-flag. Turnovers \> 40 = flag.

-   Match result enum: W \| L only. NBA 2K competitive modes do not use OTL or SOL.

-   FG%/3PT% computation: If percentage not shown directly, pipeline computes from Made/Attempted values and stores both raw counts and computed percentage.

+-------------------------------------------------------------------------------------+
| **2.5 NHL**                                                                         |
|                                                                                     |
| *schema_nhl_h2h \| schema_nhl_hut --- Team Stats --- Post-Game Team Summary Screen* |
+=====================================================================================+
+-------------------------------------------------------------------------------------+

NHL\'s post-game screen shows a clean team stats comparison. User team row identified by team name or left-side default positioning on the results layout. HUT Rivals and HUT Champions share the same field set. NHL is the only title in scope where OTL and SOL are distinct match result outcomes.

  ----------------------------------------------------------------------------------------------------------------
  **Team Stat**              **Type**     **Notes**
  -------------------------- ------------ ------------------------------------------------------------------------
  Goals Scored               Integer      User team total goals. Required.

  Goals Allowed              Integer      Opponent goals. Required. Drives match_result.

  Shots on Goal              Integer      Total shots on goal by user team. Required.

  Shots Against              Integer      Total shots faced by user team. Required.

  Save %                     Percentage   Stored as decimal 0.000--1.000. Displayed as % (e.g. 94.2%). Required.

  Power Play Goals           Integer      Goals on power play. Optional.

  Power Play Opportunities   Integer      Total PP chances. Optional.

  Penalty Kill %             Percentage   \% of opponent power plays killed. 0--100. Optional.

  Hits                       Integer      Body checks by user team. Optional.

  Blocked Shots              Integer      Shots blocked by user team. Optional.

  Faceoffs Won %             Percentage   Faceoff win percentage. 0--100. Optional.

  Final Score (user)         Integer      Required.

  Final Score (opponent)     Integer      Required.
  ----------------------------------------------------------------------------------------------------------------

-   Fraud bounds: Goals Scored \> 15 = flag. Save % outside 0.000--1.000 = auto-flag. Shots on Goal \> 80 = flag.

-   Match result enum: W \| L \| OTL \| SOL. NHL is the only sports title in scope with OTL and SOL as distinct outcomes. Capture accurately from the post-game result indicator.

-   Save % stored as decimal (0.000--1.000). Displayed to player as percentage e.g. 94.2%.

+---------------------------------------------------------------------------------+
| **2.6 MLB The Show**                                                            |
|                                                                                 |
| *schema_mlb_h2h \| schema_mlb_dd --- Team Stats --- Post-Game Box Score Screen* |
+=================================================================================+
+---------------------------------------------------------------------------------+

MLB The Show is the only innings-based title in scope. The post-game box score screen shows team batting and pitching totals for the full game. Both H2H and Diamond Dynasty Ranked Seasons and Events are in scope and share the same field set. DD Events may use shorter game formats (3 or 6 innings) --- the Innings Played field captures this naturally.

  -------------------------------------------------------------------------------------------------------------------
  **Team Stat**              **Type**   **Notes**
  -------------------------- ---------- -----------------------------------------------------------------------------
  Runs Scored                Integer    User team total runs. Required.

  Runs Allowed               Integer    Opponent runs. Required. Drives match_result.

  Hits                       Integer    Total hits by user team. Required.

  Errors                     Integer    Fielding errors committed by user team. Required.

  Home Runs                  Integer    Home runs hit by user team. Required.

  RBI                        Integer    Runs batted in --- team total. Required.

  Strikeouts (batting)       Integer    Times user batters struck out. Optional.

  Walks (batting)            Integer    Walks drawn by user team. Optional.

  Stolen Bases               Integer    Successful stolen bases by user team. Optional.

  Left on Base               Integer    Runners left on base. Optional.

  Strikeouts (pitching)      Integer    Ks thrown by user team\'s pitching. Optional.

  Walks Allowed (pitching)   Integer    Walks issued by user pitching. Optional.

  ERA (pitching)             Decimal    Earned Run Average. Stored as X.XX decimal. Optional.

  Innings Played             Decimal    Standard 9.0 regulation. Extra innings possible --- no upper cap. Optional.

  Final Score (user)         Integer    Same as Runs Scored. Required.

  Final Score (opponent)     Integer    Required.
  -------------------------------------------------------------------------------------------------------------------

-   Fraud bounds: Runs Scored \> 30 = flag. Home Runs \> 15 = flag. Hits \> 30 = flag. ERA \> 27.00 = flag (mathematical ceiling for a single game).

-   Match result enum: W \| L \| D only. Ties possible if time limits are applied in The Show competitive modes.

-   Extra innings: Innings Played allows values above 9.0 with no upper fraud bound --- extra innings are legitimate game states.

-   DD Events: Shorter game formats (3 or 6 innings) show lower run and hit totals --- the Innings Played field provides context. Fraud bounds remain the same since even a 3-inning game cannot legitimately produce 30 runs.

-   ERA: Displayed as X.XX on The Show screens. Hard floor 0.00, hard ceiling 27.00 (mathematically impossible to exceed in a single game).

  -----------------------------------------------------------------------
  **SECTION 3 --- PIPELINE EXTENSIONS FOR SPORTS GAMES**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## 3.1 Pipeline Extensions --- Sports-Specific Steps

The core 10-step OCR pipeline from OCR Stat Tracking Spec v1.0 Section 3.1 applies unchanged. The following extensions insert between Steps 4 and 6. No position detection step exists in this version.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Step**   **Extension**                  **Logic**
  ---------- ------------------------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  4A         Mode Detection                 After game_id confirmed, classify mode_id from UI elements: mode badge, screen header text, menu context, layout pattern. Threshold 0.75. Fallback: primary H2H schema + flag_reason_mode = \'mode_undetected\'.

  4B         Madden/CFB Disambiguation      Triggered only when game_id detection confidence \< 0.80 between these two titles. Uses college logos, uniform design, field markings, EA CFB watermark. If unresolved: flag_reason_mode = \'madden_cfb_ambiguous\'. Route to admin review.

  4C         User Team Row Identification   Identify the user\'s team row on the post-game box score. Priority: (1) gamertag match against player profile, (2) highlighted/colored row indicator, (3) Home team assumption. Flag if ambiguous.

  5A         Sports UI Pre-Processing       Apply sport-specific contrast filters and animated overlay suppression (confetti, trophy screens, celebration animations). Isolate scoreboard region to avoid extracting stadium/crowd UI text.

  6A         Mode-Scoped Schema Mapping     Apply SportsStatSchema matching both game_id AND mode_id. position_aware = false for all sports schemas --- no position scoping required. One flat field set per game + mode.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 3.2 Fraud Plausibility Bounds --- All Six Titles

  -------------------------------------------------------------------------------------------------------------------------------------
  **Game / Stat**             **Valid Range**   **Flag Threshold**   **Notes**
  --------------------------- ----------------- -------------------- ------------------------------------------------------------------
  Madden --- Total Yards      0 -- 1,000        \> 1,000             NFL single-game team record \~735 yds.

  Madden --- Total TDs        0 -- 15           \> 15                Covers all scoring types.

  CFB --- Total Yards         0 -- 1,100        \> 1,100             Higher ceiling than Madden for college pace.

  CFB --- Total TDs           0 -- 18           \> 18                Higher ceiling than Madden.

  EA FC --- Goals Scored      0 -- 20           \> 20                Covers FUT abuse edge cases.

  EA FC --- Possession %      0 -- 100          Outside 0--100       Hard mathematical ceiling. Auto-flag.

  EA FC --- Pass Accuracy %   0 -- 100          Outside 0--100       Hard mathematical ceiling. Auto-flag.

  NBA 2K --- Points           0 -- 200          \> 200               Team points including OT scenarios.

  NBA 2K --- FG%/3PT%/FT%     0 -- 100          Outside 0--100       Hard mathematical ceiling. Auto-flag.

  NBA 2K --- Turnovers        0 -- 40           \> 40                Generous ceiling for extreme games.

  NHL --- Goals Scored        0 -- 15           \> 15                NHL single-game record is 16.

  NHL --- Save %              0.000 -- 1.000    Outside range        Hard mathematical ceiling. Auto-flag.

  NHL --- Shots on Goal       0 -- 80           \> 80                NHL single-game record \~73 shots.

  MLB --- Runs Scored         0 -- 30           \> 30                Modern game practical ceiling.

  MLB --- Home Runs           0 -- 15           \> 15                MLB team HR record is 10. 15 covers game sim variance.

  MLB --- ERA                 0.00 -- 27.00     \> 27.00             Mathematical ceiling --- 3 ERs per batter, no outs, full inning.
  -------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 4 --- CRM FIELD EXTENSIONS**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## 4.1 New CRM Fields --- Player Profile

*These fields extend Section 4.1 of OCR Stat Tracking Spec v1.0 and are added to the player CRM profile alongside existing stat_submissions\[\] and career_stats_by_game fields.*

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Name**                  **Type**   **Source**   **Notes / Logic (CRM Fields)**
  ------------------------------- ---------- ------------ ----------------------------------------------------------------------------------------------------------------------------------------------------------
  sports_career_stats             JSON       Computed     {game_id: {mode_id: {field_key: aggregate_value}}}. Two-level hierarchy: game -\> mode. Recomputed on every approval/rejection.

  sports_peak_stats               JSON       Computed     {game_id: {mode_id: {field_key: {value, match_id, date}}}}. Best single-match value per stat. Updated on each approval if new value exceeds stored peak.

  sports_h2h_record               JSON       Computed     {game_id: {mode_id: {wins, losses, otl, sol, draws, win_pct}}}. OTL and SOL populated for NHL only --- null for all other titles.

  sports_season_totals            JSON       Computed     {game_id: {mode_id: {season_id: {field_key: cumulative_value}}}}. Season-scoped totals following AEU platform season_id_current.

  sports_mode_submissions_count   JSON       Computed     {game_id: {mode_id: count}}. Submission counts per game per mode. Powers mode tab labels and badge progress.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 4.2 Aggregation Logic

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Aggregate Type**               **Method**                                                         **Example**
  -------------------------------- ------------------------------------------------------------------ ------------------------------------------------------------------------------------
  Per-game averages                Sum / approved count for game + mode                               Madden H2H: 387.4 total yards/game \| 4.2 TDs/game \| 1.1 TOs/game across 32 games

  Season cumulative totals         Sum across all submissions in current season_id                    NBA 2K H2H Season 2: 3,847 total points \| 214 assists \| 189 turnovers

  H2H record                       Count W/L/OTL/SOL/D from match_result per game + mode              NHL HUT: 28W-10L-4OTL-2SOL \| Win %: 63.6% (44 games)

  Peak single-match                Max per field per game + mode across approved submissions          EA FC FUT: Best match --- 6 Goals \| 78% Possession \| 94% Pass Accuracy

  Percentage fields                Weighted average (weighted by games played)                        MLB DD: .298 batting average across 47 tracked games

  X/Y string fields                Sum numerators and denominators separately, recompute percentage   NBA 2K career 3PT: 847/2,134 = 39.7% across all H2H submissions

  Inverse stats (lower = better)   Min tracked as peak (errors, turnovers, runs allowed, etc.)        MLB: Best single-game Errors = 0. Best ERA = 0.00
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 5 --- FRONTEND EXTENSIONS**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## 5.1 Review Form Extensions --- Sports Games

The StatReviewForm component from OCR Stat Tracking Spec v1.0 Section 5.4 requires three additions for sports games. No position selector is needed.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **UI Element**                  **Behavior**
  ------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Mode Selector                   Dropdown above stat fields. Shows when mode_detected = false OR player taps \'Wrong mode?\'. Lists in-scope modes for the detected game. Changing mode triggers schema re-mapping (no new OCR run). Detected mode shown as default with confidence indicator.

  Match Result Selector           W / L / OTL / SOL / D pill selector at top of form. OTL and SOL pills rendered only for NHL --- hidden for all other titles. Pre-filled from extraction. Required --- submit blocked until confirmed.

  Score Fields                    Two adjacent editable inputs: user score / opponent score. Pre-filled from extraction. Player verifies before submitting.

  [No Position Selector]{.mark}   [Not rendered for any sports schema. position_aware = false gates this component from rendering for all sports titles.]{.mark}
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 5.2 Career Stats Panel --- Sports Game Display

**Web App**

-   Top level: Game tabs --- one per sports title with tracked submissions (NBA 2K \| Madden \| EA FC \| NHL \| CFB \| MLB). Empty titles not shown.

-   Second level: Mode sub-tabs within each game (H2H \| Ultimate Team \| Pro-Am \| Play Now \| etc.). Empty modes not shown.

-   Stat grid: StatTile components for per-game averages. Season totals panel below averages. H2H record card above stat grid.

-   Peak records: Horizontal strip of highlight cards --- best single-match value per stat with match score and date.

-   Mode tab shows match count label: \'X matches tracked\'.

**Telegram Mini App**

-   Vertical accordion per game. Collapsed state shows game name + match count. Tap to expand.

-   Mode pills within each accordion section. Tap to switch active mode view.

-   Stats as StatTile stack --- no tables, no charts. 4 tiles per row maximum.

-   H2H record as single compact tile: \'28W 10L 63.6%\' or \'28W 10L 4OTL 2SOL\' for NHL.

-   Peak records as horizontal scroll card strip below stat tiles.

## 5.3 Hook Updates

*Extends usePlayerStats() from OCR Stat Tracking Spec v1.0 Section 5.1.*

  -------------------------------------------------------------------------------------------------
  **New Return Field**            **Type**              **Powers**
  ------------------------------- --------------------- -------------------------------------------
  sports_career_stats             JSON Object           Career stats tab content per game + mode.

  sports_peak_stats               JSON Object           Peak record highlight cards.

  sports_h2h_record               JSON Object           H2H record card per game + mode.

  sports_season_totals            JSON Object           Season totals panel.

  sports_mode_submissions_count   JSON Object           Mode tab match count labels.
  -------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **SECTION 6 --- EXECUTION TRACKER**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## 6.1 Backend --- Execution Tracker

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       **Feature**                                       **Implementation Notes**
  ---- ------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ☐    Mode detection classifier (all 6 titles)          Classify mode_id from post-game screen for all in-scope modes. Threshold 0.75. Fallback to H2H schema + flag.

  ☐    Madden/CFB disambiguation classifier              Activated at game_id detection confidence \< 0.80 between these titles. College logos, uniform design, field markings, EA CFB watermark. flag_reason_mode = \'madden_cfb_ambiguous\' if unresolved.

  ☐    User team row identification                      Priority: (1) gamertag match, (2) highlighted row, (3) Home team assumption. Flag if ambiguous.

  ☐    Sports UI pre-processing filters                  Contrast enhancement, animated overlay suppression, scoreboard region isolation. Per-game filter tuning for each title\'s visual style.

  ☐    SportsStatSchema registry (12 schemas)            2 schemas per game (H2H + ranked mode). Registered alongside esports schemas. Versioned. API: GET /schemas/:game_id/:mode_id

  ☐    Sports plausibility bounds enforcement            All bounds from Section 3.2 implemented in a separate sports bounds config file. Applied in Step 9 (fraud pre-check) of the core pipeline.

  ☐    SportStatSubmission record extension              mode_id, mode_detected, match_result, score_user, score_opponent, flag_reason_mode written on every sports game submission.

  ☐    sports_career_stats aggregation (game -\> mode)   Two-level hierarchy. Per-game averages + season totals + H2H record derived from approved submissions. Recomputed on every approval/rejection.

  ☐    sports_peak_stats tracking                        Compare incoming approved values against stored peaks. Update if new value is better. Inverse stats (errors, TOs, runs allowed) track minimums as peaks.

  ☐    sports_h2h_record computation                     Derive W/L/OTL/SOL/D and win_pct from match_result. OTL/SOL populated for NHL only --- null for all other titles.

  ☐    X/Y string field parsing                          Parse FG Made/Attempted, 3PT, FT, Third Down Conversions as X/Y strings. Store raw string + computed percentage separately.

  ☐    NHL Save % normalization                          Store as decimal 0.000--1.000. Display as percentage e.g. 94.2%.

  ☐    MLB ERA normalization                             Extract as X.XX decimal. Validate 0.00--27.00. Auto-flag above 27.00.

  ☐    Admin queue: mode flag reason display             \'mode_undetected\' and \'madden_cfb_ambiguous\' shown as distinct flag types --- visually separate from fraud flags.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 

## 6.2 Frontend --- Execution Tracker

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       **Feature**                                          **Implementation Notes**
  ---- ---------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ☐    Mode Selector in StatReviewForm                      Dropdown above stat fields. Changing mode triggers field list re-render via schema re-mapping. No new OCR run.

  ☐    Match Result selector (W/L/OTL/SOL/D)                Pill selector. OTL + SOL pills only for NHL --- hidden for all other titles. Required field.

  ☐    Score fields in review form                          Two adjacent editable inputs. Pre-filled. Player verifies.

  ☐    No position selector                                 Confirm in code that position_aware = false check gates this component from rendering entirely for sports schemas.

  ☐    Career Stats panel --- game + mode tabs (web)        Top-level game tabs. Mode sub-tabs within. StatTile grid for averages. Season totals below. H2H record card above grid. Peak highlight cards.

  ☐    Career Stats panel --- accordion layout (Telegram)   Vertical accordion per game. Mode pills within. StatTile stack. H2H compact tile. Peak records horizontal scroll.

  ☐    NHL OTL/SOL H2H record display                       NHL H2H record card shows W-L-OTL-SOL. All other titles show W-L (plus D for EA FC and MLB).

  ☐    Peak stat highlight cards                            Per game + mode. Stat name, value, match score, date. Horizontal scroll on both web and Telegram.

  ☐    Season totals panel                                  Cumulative totals for current season_id. Season label shown. Display resets on season change.

  ☐    Sports milestone badges in collection grid           All 9 sports badges displayable with correct rarity color. Badge unlock animation fires on trigger.

  ☐    usePlayerStats() extended returns                    5 new sports fields: sports_career_stats, sports_peak_stats, sports_h2h_record, sports_season_totals, sports_mode_submissions_count.

  ☐    Admin queue: sports flag reason display              mode_undetected and madden_cfb_ambiguous shown with distinct icon and label. Visually differentiated from fraud flags --- these are detection failures, not integrity issues.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

+------------------------------------------------------------------------------------------------------------------+
| AEU OCR Stat Tracking --- Sports Games Addendum v2.0 \| February 2026 \| Addendum to OCR Stat Tracking Spec v1.0 |
|                                                                                                                  |
| Confidential --- Internal Use Only --- Not for Distribution                                                      |
+==================================================================================================================+
+------------------------------------------------------------------------------------------------------------------+


---

# PART 4: Member Workflows — Monetization, Marketplace, Token Utilities, Gamification

**AEU MEMBER WORKFLOWS**

Monetization \| Marketplace \| Token Utilities \| Gamification

Internal Product & Strategy Document \| Amateur Esports Union \| Most Vicious Players LLC

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Document Structure**                                                                                                                                                                                                                                                                                                    |
|                                                                                                                                                                                                                                                                                                                           |
| This document is organized into four parts. Part 1 covers the core member workflows and monetization logic for all user types. Part 2 covers the marketplace layer embedded within each workflow. Part 3 maps the token utility system across all roles. Part 4 defines the unified gamification and social architecture. |
+===========================================================================================================================================================================================================================================================================================================================+
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

  --------------------------------------------------------------------------------------------------------------------------------
  **PART 1Workflows + Monetization**   **PART 2Marketplace Layer**   **PART 3Token Utilities**   **PART 4Gamification + Social**
  ------------------------------------ ----------------------------- --------------------------- ---------------------------------

  --------------------------------------------------------------------------------------------------------------------------------

**PART 1**

**Member Workflows & Monetization**

+-----------------------------------------------------------------------+
| **USER TYPE 01**                                                      |
|                                                                       |
| **Player Workflow**                                                   |
|                                                                       |
| Free + Player Premium \| 13 Steps                                     |
+=======================================================================+
+-----------------------------------------------------------------------+

The player workflow is the core revenue engine of the platform. Every step in the funnel either generates revenue directly or sets up a future revenue event. The workflow begins at discovery and loops back at re-entry --- the objective is to keep winnings circulating inside the platform rather than withdrawing them.

## **Core Workflow Steps**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **\#**   **Action**                         **Features**                                                       **Revenue Logic**
  -------- ---------------------------------- ------------------------------------------------------------------ --------------------------------------------------------------------------------------------------------------------------------------------------------
  **1**    Landing / Discovery                Telegram Mini App sharing, affiliate/referral system               Lower CAC via organic shares. Referral attribution enables future performance payouts. Sponsored tournament discovery placement sold to sponsors.

  **2**    Account Creation / Login           Telegram authentication, identity integration                      Higher conversion rate → more paid entrants. Player Premium subscription conversion opportunity at this step.

  **3**    Membership Selection               Membership tiering, token eligibility controls                     Subscription revenue starts at checkout. Token bundle sold here. Premium add-ons: extra stats, priority access.

  **4**    Wallet Setup / Funding             Fiat wallet (USD), dual-rail payments, optional crypto toggle      Funded wallet increases repeat entry likelihood. Card processing fee optimization. Crypto rail expands market reach. Wallet-to-wallet transfer fees.

  **5**    Tournament Browsing                Mini App UI, analytics surfaces, sponsor placements                Sponsored placement inventory sold here. Featured tournament placements sold to TOs. Premium gating drives subscription upgrades.

  **6**    Tournament Registration            Wallet payment, token spend option, membership gating              Entry fee rake triggers here (core revenue). Token spend triggers token economy margin. Upsell: Premium upgrade, Org/Team join.

  **7**    Payment / Checkout                 Dual-rail payments, wallet, dispute workflow baseline              Rake already triggered --- now protect margin via reduced payment failure. Minimum funding amount. Reduced chargebacks via confirmation flows.

  **8**    Pre-Match / Check-In               Telegram bot notifications, check-in system                        Reduces no-shows → prevents refunds → protects net revenue. Micro-upsells: last-minute token purchase, wallet top-up. Stat challenges for bonus drops.

  **9**    Match Play + Score Reporting       Manual reporting or OCR automation (Phase IV), dispute workflows   OCR reduces staffing → margin expansion. Lower dispute rate reduces refunds. Higher reliability increases retention.

  **10**   Streaming / Viewing                Telegram native streaming, Twitch/YouTube/Kick embeds              Sponsorship overlay inventory monetized. Advertisement commercial breaks. Premium viewing paywalled. Streams drive more entrants.

  **11**   Post-Match Rewards + Progression   Tokens, achievement badges, analytics dashboards                   Tokens encourage re-entry and spending. Badges increase status retention → raises LTV. Premium analytics upsell. Sponsor placement.

  **12**   Winnings Credited                  Automated prize distribution, wallet credit                        Wallet retention increases re-entry rate → recycled rake. Auto-suggest relevant next tournament based on entry history.

  **13**   Withdraw or Reuse Funds            Wallet management, settlement workflows                            Withdrawal friction increases in-platform reuse. Premium membership reduces withdrawal friction (subscription driver). W-9 after \$600 winnings.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Player Revenue Summary**

  -------------------------------------------------------------------------------------------------------------
  **Revenue Type**        **Trigger Point**                    **Notes**
  ----------------------- ------------------------------------ ------------------------------------------------
  **Subscription**        Step 3 --- Membership selection      Player Premium --- immediate recurring revenue

  **Entry Fee Rake**      Step 6 --- Tournament registration   Core transaction revenue --- every paid entry

  **Token Economy**       Step 6, 8, 11                        Token spend margin + breakage

  **Sponsor Placement**   Steps 1, 5, 10, 11                   Discovery, browse, stream overlay, post-match

  **Processing Fees**     Step 7 --- Checkout                  Margin optimization on payment processing

  **Premium Analytics**   Step 11 --- Post-match               Upsell on performance data

  **Wallet Reuse**        Step 12-13                           Recycled rake from retained winnings
  -------------------------------------------------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **USER TYPE 02**                                                      |
|                                                                       |
| **Organization Premium Workflow**                                     |
|                                                                       |
| Org Admin + Teams \| 13 Steps                                         |
+=======================================================================+
+-----------------------------------------------------------------------+

Organizations operate at scale --- managing multiple teams, bulk tournament entries, and internal player incentive systems. Their workflow is both operational and commercial. The org is simultaneously a platform customer and a revenue amplifier, driving player entry volume and sponsor campaign reach.

## **Core Workflow Steps**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **\#**   **Action**                                 **Features**                                              **Revenue Logic**
  -------- ------------------------------------------ --------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------
  **1**    Org Discovery / Lead Capture               Affiliate/referrals, featured org placement               Enterprise lead pipeline begins. Referral attribution enables org sales commissions.

  **2**    Org Account Creation                       Org Premium product, verification badge option            Org Premium subscription triggers immediately. Verification badge / featured listing upsell.

  **3**    Choose Tier + Capacity                     Tiered Org Premium, role-based permissions                Subscription + optional annual prepay cashflow. Add-ons: extra teams, seats, analytics, badges.

  **4**    Branding + Public Profile Setup            Branding layer, org badge system                          Paid profile customization. Premium storefront. Verified org upsell. Sponsor-facing presence increases deal conversion.

  **5**    Roles + Permissions                        Permission system                                         Multi-admin is a tiered feature (seat-based pricing). Finance controls as premium tier upsell.

  **6**    Create Teams                               Multi-team management                                     Team cap exceeded → per-team add-on revenue. Team branding bundles as add-ons.

  **7**    Add Players to Teams                       Identity layer, membership gating                         Bulk invites create Player Premium upsell opportunities. Org can sponsor Player Premium seat bundles.

  **8**    Fund Wallet + Allocate Budgets             Fiat wallet, dual-rail payments, optional crypto toggle   Larger wallet deposits drive more rake volume. Crypto rail unlocks international team participation.

  **9**    Bulk Tournament Discovery + Registration   Mini App / web UI, bulk entry, wallet payment             Rake triggers on every team entry. Bulk entry as premium tier feature. Token spend as entry method.

  **10**   Team Check-In + Match Execution            Bot notifications, OCR, dispute workflows                 Reduced no-show refunds protects margin. OCR reduces staffing costs.

  **11**   Org Performance Analytics                  Analytics dashboard                                       Premium analytics upsell. Recruiter/scouting visibility add-on.

  **12**   Prize Settlement + Redistribution          Automated prize distribution, wallet ledger               Wallet retention drives re-entry. Org keeps winnings circulating. Players must pre-agree to prize split parameters.

  **13**   Sponsor Integration (Org-level)            Sponsor marketplace, badges, tokens, streaming overlays   Sponsor fees. Sponsored badge drops tied to org performance. Token multipliers as paid campaigns.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Org Revenue Summary**

  ----------------------------------------------------------------------------------------------------------------------------------
  **Revenue Type**               **Trigger Point**                       **Notes**
  ------------------------------ --------------------------------------- -----------------------------------------------------------
  **Org Premium Subscription**   Step 2-3 --- Account + tier selection   Recurring subscription --- tiered by teams/seats/features

  **Add-On Upsells**             Steps 3-7                               Extra teams, seats, analytics, badges, branding

  **Bulk Entry Rake**            Step 9 --- Tournament registration      Rake multiplied across all team entries

  **Seat Bundles**               Step 7 --- Player management            Org sponsors Player Premium seats for roster members

  **Sponsor Campaign Fees**      Step 13 --- Sponsor integration         Org-level sponsored badges, token multipliers, overlays

  **Token Economy**              Steps 8-9                               Org wallet token purchases + bulk entry token spend
  ----------------------------------------------------------------------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **USER TYPE 03**                                                      |
|                                                                       |
| **Tournament Organizer Workflow**                                     |
|                                                                       |
| TO Membership \| 10 Steps                                             |
+=======================================================================+
+-----------------------------------------------------------------------+

Tournament organizers are the supply side of the AEU marketplace. Without quality events running consistently, there is no rake and no player activity. The TO workflow is designed to make event creation frictionless and revenue-generating at every step, while building the reputation infrastructure that brings players back to their events repeatedly.

## **Core Workflow Steps**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **\#**   **Action**                        **Features**                                            **Revenue Logic**
  -------- --------------------------------- ------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------------
  **1**    TO Discovery / Signup             TO membership product                                   TO subscription triggers here. Free TO funnels into upsell.

  **2**    Verification + Compliance Setup   Identity verification, payout setup, rules acceptance   Verified TO badge as paid upgrade. Compliance gating protects merchant accounts.

  **3**    Membership Tier Selection         TO membership tiers                                     Subscription revenue triggers. Tier-gated access to sponsor slots, featured placement, analytics.

  **4**    Tournament Creation               Mini App UI, API integration                            Featured tournament listing upsell. Sponsor slots created and priced here. Token-gated tournaments force token purchases.

  **5**    Prize Pool Funding / Escrow       Wallet escrow logic, automated settlement rails         Enables higher entry fee pricing due to trust. Reduces unpaid prize disputes.

  **6**    Publish Tournament                Discovery surfaces, sponsor placements                  Featured placement revenue (if purchased). Sponsorship inventory becomes sellable.

  **7**    Player / Org Registrations        Wallet payments, dual rail, tokens                      Entry fee rake per entry (core). Token usage triggers token revenue. Payment optimization reduces lost sales.

  **8**    Run Tournament Operations         Bot notifications, OCR, dispute workflows               Lower refunds/chargebacks protect margin. OCR reduces staffing costs.

  **9**    Streaming Activation              Telegram streaming + embeds                             Sponsor overlays monetized. Premium viewing gated by membership/token spend.

  **10**   Settlement                        Automated prize distribution, refunds/disputes          Automation reduces operational expense. Trust increases repeat tournament volume.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **TO Revenue Summary**

  -------------------------------------------------------------------------------------------------------------------------
  **Revenue Type**                **Trigger Point**                **Notes**
  ------------------------------- -------------------------------- --------------------------------------------------------
  **TO Subscription**             Step 3 --- Tier selection        Recurring membership --- tiered by features and access

  **Featured Listing**            Step 4 --- Tournament creation   Upsell for discovery placement boost

  **Verified Badge**              Step 2                           One-time or recurring paid credential upgrade

  **Rake (via player entries)**   Step 7 --- Registrations         TO events generate AEU rake on every entry

  **Sponsor Placement**           Steps 6-9                        Slot inventory sold within TO events

  **Premium Viewing**             Step 9 --- Streaming             Token or subscription gated stream access
  -------------------------------------------------------------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **USER TYPE 04**                                                      |
|                                                                       |
| **Sponsor Workflow**                                                  |
|                                                                       |
| Sponsor Account \| 5 Steps                                            |
+=======================================================================+
+-----------------------------------------------------------------------+

Sponsors are the commercial fuel of the AEU ecosystem. They fund token drops, badge campaigns, and multiplier events that drive player engagement --- while AEU charges for placement and eventually takes a share of the GMV their campaigns generate. The sponsor relationship is designed to start simple and deepen over time as ROI data accumulates.

## **Core Workflow Steps**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **\#**   **Action**              **Features**                                               **Revenue Logic**
  -------- ----------------------- ---------------------------------------------------------- -----------------------------------------------------------------------------------------------------------
  **1**    Sponsor Onboarding      Sponsor marketplace, analytics foundation                  Platform fee / subscription for sponsor account. Premium access tiers to placements and data.

  **2**    Choose Placement Type   Badges, tokens, streaming, tournament listings, surveys    Sponsorship fees trigger here. Higher pricing for org-level multi-team exposure.

  **3**    Campaign Activation     Mini App placements, website placements, stream overlays   CPM / flat fee / performance fee begins. Creatives go live across placement inventory.

  **4**    Reward Mechanics        Tokens, badges                                             Paid incentive campaigns. Tokens create engagement loops that increase entry fees indirectly.

  **5**    Reporting + Renewal     Advanced analytics dashboard                               Premium reporting upsell triggers. Data-driven ROI increases renewal rates --- recurring sponsor revenue.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Sponsor Revenue Summary**

  ---------------------------------------------------------------------------------------------------------------------
  **Revenue Type**              **Trigger Point**               **Notes**
  ----------------------------- ------------------------------- -------------------------------------------------------
  **Sponsor Account Fee**       Step 1 --- Onboarding           Platform subscription or one-time access fee

  **Placement Fees**            Step 2-3 --- Campaign           CPM, flat fee, or performance-based pricing

  **Token Campaign Purchase**   Step 4 --- Reward mechanics     Sponsor buys token pool for player drops

  **Badge Creation Fee**        Step 4                          Custom sponsor badge for player collection

  **Premium Reporting**         Step 5 --- Analytics            Upsell on advanced attribution and ROI data

  **Revenue Share (Phase 3)**   Post-launch --- GMV milestone   Percentage of product transactions routed through AEU
  ---------------------------------------------------------------------------------------------------------------------

**PART 2**

**Product Marketplace Layer**

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Core Principle**                                                                                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                                                                     |
| Without the marketplace: AEU = tournament monetization platform. With the marketplace: AEU = competitive commerce infrastructure. The marketplace is layered across every member type but kept operationally separate from the competitive engine. It activates at specific emotional and high-intent moments within each workflow. |
+=====================================================================================================================================================================================================================================================================================================================================+
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

  --------------------------------------------------------------------------------------------------------------------------------------
  PlayersEmotional buying   OrgsBulk buying + affiliate   TOsEvent-based pop-up commerce   SponsorsFull funnel: visibility → GMV split
  ------------------------- ----------------------------- -------------------------------- ---------------------------------------------

  --------------------------------------------------------------------------------------------------------------------------------------

## **Player Marketplace Layer**

The marketplace activates at specific emotional moments within the player workflow --- discovery, registration, streaming, post-match rewards, and winnings credit. Each is a distinct buying intent window.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Touchpoint**                **Marketplace Placement / Feature**                                                                                    **Purpose**
  ----------------------------- ---------------------------------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------
  **Discovery**                 Featured gear for selected games. Sponsored placements. Merit-based gear recommendations.                              Immediate sponsor visibility. Track click intent. Commerce attribution before registration.

  **Account Creation**          Capture gear preferences and brand affinity. Personalize storefront feed.                                              Higher conversion probability. Sponsor targeting precision. Data foundation for revenue share.

  **Membership Selection**      Premium-only discounts. Early access drops. Token cashback. Loyalty program. Tokens bundled with premium.              Subscription becomes commerce-enhanced. Sponsors pay premium for premium placement.

  **Wallet Setup**              Wallet usable for product purchases. Tokens redeemable for discounts. Sponsor-funded promo credits.                    Reduce checkout friction. Keep funds circulating in ecosystem.

  **Tournament Browsing**       Game-specific gear recommendations. Sponsored tournament kits. Shoppable leaderboard banners.                          Convert competitive intent into buying intent. Attribute tournament → purchase conversion.

  **Tournament Registration**   Limited event merch. Sponsor bundles. \'Buy before event\' prompts.                                                    High emotional conversion window. Scarcity-based purchase triggers.

  **Streaming / Viewing**       Clickable live overlays. Stream-exclusive discount codes. Gear displayed on top players. Token drops during streams.   Convert spectators to buyers. Tie stream visibility to sponsor ROI.

  **Post-Match Rewards**        Token → discount redemption. Performance-based gear suggestions. Sponsor badge unlocks.                                Emotional buying moment. Increase GMV through reward psychology.

  **Winnings Credited**         \'Use winnings toward marketplace.\' Bonus token multiplier if applied to product purchase.                            Reduce withdrawals. Increase transaction volume.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Organization Marketplace Layer**

Marketplace for orgs is bulk and affiliate-driven. The org becomes a commerce amplifier --- sponsor visibility scales across all teams simultaneously.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Touchpoint**          **Marketplace Placement / Feature**                                                    **Purpose**
  ----------------------- -------------------------------------------------------------------------------------- ------------------------------------------------------------------------------
  **Org Profile Setup**   Org storefront page. Co-branded sponsor placement. Affiliate tracking per org.         Org becomes commerce amplifier. Sponsor visibility scales across teams.

  **Team Creation**       Bulk apparel packages. Team sponsor bundles. Equipment kits.                           High-ticket purchases. Multi-player transaction lift.

  **Adding Players**      Player affiliate tracking. Discount codes per team. Sponsor referral leaderboard.      Internal incentive to drive product sales. Sponsor ROI attribution per team.

  **Org Dashboard**       Merch conversion analytics. Affiliate revenue tracking. Product performance reports.   Org becomes sponsor sales partner. Enables rev share discussions.

  **Prize Settlement**    Allocate winnings toward org gear. Reward top players with sponsor products.           Recycle funds into commerce. Encourage internal team identity.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Tournament Organizer Marketplace Layer**

Each tournament becomes a temporary storefront. The TO event lifecycle --- creation, promotion, live operations, post-event --- maps directly to a commerce lifecycle.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Touchpoint**            **Marketplace Placement / Feature**                                                                  **Purpose**
  ------------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------
  **Tournament Creation**   Featured event products storefront. Sponsor product placement modules. Official event merchandise.   Every event becomes a commerce channel. Sponsor placement becomes sellable inventory.

  **Pre-Event Promotion**   Email + Telegram gear promotion. Limited event-branded drops.                                        Pre-event revenue before tournament starts.

  **During Tournament**     Stream overlays. Bracket page product placements. Live discount unlocks.                             Capture live emotional engagement at peak attention moment.

  **Post-Tournament**       Champion edition merch. Event recap product links.                                                   Extend revenue lifecycle beyond event day.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Sponsor Marketplace Layer --- Three Phases**

The sponsor marketplace evolves across three phases aligned with AEU\'s platform maturity. Each phase unlocks more commercial depth.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Phase**                      **Marketplace Role**                                                                                   **Purpose**
  ------------------------------ ------------------------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------------
  **Phase 1Visibility**          Brand placements. Sponsor dashboard (impressions + CTR). Tiered exposure levels.                       Immediate monetization without fulfillment complexity.

  **Phase 2API Order Routing**   In-platform product browsing. Order intent routed to sponsor backend. Confirmation returned to user.   AEU does not handle inventory. Sponsor handles fulfillment. AEU monetizes transaction flow.

  **Phase 3Revenue Share**       GMV tracking. Conversion attribution. Real-time ROI reporting.                                         Enable percentage-based revenue agreements. Transition sponsors into enterprise contracts.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**PART 3**

**Token Utility System**

+----------------------------------+-----------------------+------------------------------------+
| **The Token IS**                 | **The Token IS NOT**  | **Strategic Effect**               |
+==================================+=======================+====================================+
| Access currency                  | An investment         | Reduces cash withdrawals           |
|                                  |                       |                                    |
| Incentive currency               | A profit expectation  | Increases in-platform reuse        |
|                                  |                       |                                    |
| Reward currency                  | A speculative asset   | Increases session frequency        |
|                                  |                       |                                    |
| Discount currency                |                       | Increases conversion velocity      |
|                                  |                       |                                    |
| Governance signal                |                       | Enables sponsor-funded engagement  |
|                                  |                       |                                    |
| Liquidity retention tool         |                       | Drives gamified loyalty            |
|                                  |                       |                                    |
| Sponsor-funded engagement engine |                       | Builds measurable internal economy |
+----------------------------------+-----------------------+------------------------------------+

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **The Most Powerful Part**                                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                    |
| When done correctly, tokens turn emotional peaks --- wins, losses, clutch moments --- into purchasing behavior without forcing cash friction. That is the real utility. The token is the bridge between competitive feeling and commercial action. |
+====================================================================================================================================================================================================================================================+
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## **Player Token Utilities**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                      **Mechanic**                                                        **Strategic Effect**
  -------------------------------- ------------------------------------------------------------------- ------------------------------------------------------------------------
  **Hybrid Entry**                 Pay \$10 OR \$8 + 20 tokens --- player chooses                      Drives token circulation. Reduces cash friction on tournament entries.

  **Premium Tournament Access**    High-level tournaments require minimum token threshold to enter     Incentivizes token accumulation. Filters for engaged players.

  **Performance Rewards**          Tokens for wins, streaks, MVP --- multipliers sponsored by brands   Behavioral reinforcement at peak emotional moment.

  **Loyalty Rewards**              Daily check-ins, participation streaks                              Daily return trigger. Mirrors Snapchat/Duolingo streak mechanics.

  **Challenge Rewards**            Solo stat challenges. Team performance challenges.                  Extends engagement beyond tournament outcomes.

  **Seasonal Leaderboard Bonus**   End-of-season token distributions by tier                           Long-arc retention anchor across 90-day competitive season.

  **Token → Product Discount**     Redeem tokens for % off sponsor gear in marketplace                 Commerce gateway --- tokens become purchasing power.

  **Token Cashback**               Earn tokens when purchasing sponsor products                        Bidirectional marketplace loop --- spending creates earning.

  **Token-Exclusive Drops**        Limited merch purchasable with tokens only                          Scarcity mechanic --- creates urgency spend events.

  **Premium Viewing Unlock**       Pay tokens to watch ad-free matches                                 Micro-spend with high perceived value.

  **Token Tips to Players**        Micro-tipping during live streams                                   Live economy layer --- drives stream viewership and engagement.

  **Token Retention Incentive**    Bonus tokens if winnings stay in wallet                             Reduces cash withdrawals --- keeps funds circulating.

  **Withdrawal Friction**          Token bonus if funds reused instead of withdrawn                    Behavioral nudge toward in-platform reuse over cash-out.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Organization Token Utilities**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                             **Mechanic**                                                        **Strategic Effect**
  --------------------------------------- ------------------------------------------------------------------- ---------------------------------------------------------------------------
  **Token Threshold for Features**        Maintain minimum token balance for advanced org analytics           Ties feature access to token holding --- creates demand floor.

  **Token-Based Team Entry**              Org bulk token entry across multiple teams                          B2B token volume channel --- orgs purchase tokens wholesale.

  **Internal Token Distribution**         Org distributes tokens to players as performance bonuses            Internal incentive system --- improves player retention within org.

  **Org Token Multiplier Campaigns**      Sponsor-funded token boosts for org-branded tournaments             Sponsor pays --- org benefits --- player earns. Three-way value creation.

  **Bulk Purchase Discount via Tokens**   Use tokens for discounts on bulk marketplace orders                 High-ticket purchase incentive --- drives org-level GMV.

  **Token-Based Sponsorship Boost**       Org locks tokens to unlock higher sponsor exposure tier (staking)   Phase 2 mechanic --- ties token holding to commercial outcome.

  **Org Stake Signal**                    Holding tokens unlocks priority sponsor matchmaking                 Token balance becomes a commercial credential for the org.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Tournament Organizer Token Utilities**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                     **Mechanic**                                                         **Strategic Effect**
  ------------------------------- -------------------------------------------------------------------- -------------------------------------------------------------------------
  **Token-Gated Tournament**      Entry requires minimum token balance --- toggle on/off per event     Forces token purchases for high-demand events. Filters engaged players.

  **Token Prize Pools**           Sponsors fund token-based prize bonuses on top of cash prizes        Larger perceived prize pool → more entries → more rake.

  **Early Registration Reward**   Token bonus for first X% of registrants --- funded by TO wallet      Drives early fill --- reduces last-minute uncertainty for TOs.

  **Partial Refund in Tokens**    Refunds issued partially in tokens to retain liquidity on platform   Keeps funds in ecosystem even when events fail.

  **Visibility Boost Spend**      TO spends tokens to boost tournament listing in discovery feed       Token spend directly tied to business outcome --- fills events.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Sponsor Token Utilities**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                            **Mechanic**                                                            **Strategic Effect**
  -------------------------------------- ----------------------------------------------------------------------- --------------------------------------------------------------------------------------
  **Sponsor-Funded Token Drops**         \'Win 50 tokens sponsored by X brand\' --- players earn, sponsor pays   Sponsor funds the engagement loop --- AEU charges for token pool + placement.

  **Token Multiplier Campaigns**         Buy product → 2x token earnings (time-limited)                          Sponsor pays to activate --- drives commerce and platform engagement simultaneously.

  **Token Badge Unlock**                 Sponsor badge requires token + specific action to unlock                Behavioral targeting --- badge only earnable by engaged, action-taking players.

  **Token-Exclusive Product Sales**      Certain drops require token payment --- not cash                        Forces token accumulation before purchase --- increases platform engagement.

  **Sponsor Purchases Token Pool**       B2B token acquisition to fund player campaigns                          Direct token revenue from sponsor side --- predictable B2B income stream.

  **Sponsor Locks Tokens for Bonuses**   MVP tournament bonus funded and locked in tokens                        Sponsor capital committed upfront --- reduces cancellation risk.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Platform-Level Token Utilities**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                           **Mechanic**                                                                **Strategic Effect**
  ------------------------------------- --------------------------------------------------------------------------- ----------------------------------------------------------------------
  **Daily Login Rewards**               Tokens for daily check-in --- streak multiplier activates at 7 days         Primary DAU driver. Low cost, high behavioral impact.

  **Weekly Active Bonus**               Minimum 1 tournament entry per week earns weekly bonus                      Drives sustained weekly engagement. Protects rake volume.

  **Re-Entry Bonus**                    Tokens earned for re-entering a tournament within 24 hours of elimination   Converts loss into re-engagement trigger --- lifts rake directly.

  **Profile Complete Reward**           One-time token bonus for completing full player profile                     Data collection with built-in incentive. Improves sponsor targeting.

  **Feedback Reward**                   Token reward for post-tournament survey completion                          Product data with incentive. Reduces survey abandonment.

  **Referral Bonus**                    Token reward when referred player enters first paid tournament              Ties referral reward to revenue event --- not just registration.

  **Partial Refund in Tokens**          Refunds issued as token credit with optional bonus vs full cash             Retains liquidity in platform ecosystem even during event failures.

  **Event Cancellation Compensation**   Cancelled event compensated in tokens rather than pure cash refund          Protects cash flow. Players remain in ecosystem.

  **Token Burn Events**                 Periodic burn events for special drops (blockchain --- future phase)        Controls supply inflation in Web3 layer. Creates scarcity events.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Marketplace-Specific Token Utilities**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Utility**                                  **Mechanic**                                                          **Strategic Effect**
  -------------------------------------------- --------------------------------------------------------------------- -----------------------------------------------------------------------------
  **Token Cashback on GMV**                    Earn tokens proportional to marketplace spend                         Bidirectional loop --- spending creates earning --- increases GMV velocity.

  **Token-Based Flash Sales**                  Token-discounted items available for 24-48 hours only                 Urgency spend cycle. Drives app opens and return sessions.

  **Token Unlock for Limited Drops**           Exclusive drops claimable with tokens --- limited quantity            Scarcity + urgency. Drives token accumulation toward specific goals.

  **Token Staking for Early Access**           Hold token balance to unlock early access to drops and registration   Demand for token holding. Rewards long-term platform participants.

  **Influencer / Player Affiliate Rewards**    Hybrid: tokens + cash for affiliate conversions                       Flexible reward structure --- tokens for engagement, cash for conversions.

  **Org Affiliate Rewards**                    Hybrid: tokens + cash for org-driven product referrals                Orgs become commerce partners. Affiliate layer at org scale.

  **Spend \$100 → Get 500 Tokens**             Commerce-to-token conversion at defined rates                         Encourages larger individual transactions. Rewards high-spend users.

  **Hold X Tokens → Unlock Tiered Discount**   Token balance unlocks permanent discount tiers                        Incentivizes token accumulation and holding. Reduces churn.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**PART 4**

**Unified Gamification + Social Architecture**

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Design Principle**                                                                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                   |
| Each user type needs its own distinct gamification layer --- different progression bars, different badge logic, different token incentives, different dashboard experience. We layer on top of this: social visibility, token economics, marketplace loops, Twitch-style live reinforcement, and cross-user system intersections. |
+===================================================================================================================================================================================================================================================================================================================================+
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## **Twitch-to-AEU Mechanic Mapping**

Twitch built one of the most effective engagement systems in digital media. AEU adapts and upgrades each mechanic by anchoring it to competitive performance rather than passive viewing.

  -------------------------------------------------------------------------------------------------------------------------------------------------------
  **TWITCH MECHANIC**                                       **AEU EQUIVALENT**
  --------------------------------------------------------- ---------------------------------------------------------------------------------------------
  Channel Points --- earned passively by watching           AEU Tokens --- earned actively by competing, entering, and engaging

  Sub Badges --- tenure visible to community                Rank Badge + Loyalty Tier --- skill AND tenure on display

  Hype Train --- communal multiplier when activity spikes   Momentum Surge --- token multiplier triggered by registration + activity spikes

  Bits --- micro-currency tipped to streamers               Token Tips --- sent to players during live match streams

  Drops --- rewards for watching specific streams           Watch-to-Earn --- tokens for watching featured tournament streams

  Predictions --- spend points to call outcomes             Token Predictions --- spend tokens on match and tournament results - Premium feature to win

  Sub emotes --- visible status in chat                     Profile frames and badges --- status on leaderboards, brackets, and stream chat

  Raids --- send audience to another channel                Org Challenges --- cross-org competition for community token drops
  -------------------------------------------------------------------------------------------------------------------------------------------------------

## **Player Gamification System**

Emotionally rich, recognition-driven, and socially visible. Players are motivated by recognition, progress, competition, and belonging. The social layer amplifies all four.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Mechanic**                              **What It Does**                                                                                   **Why It Works For This User**
  ----------------------------------------- -------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------
  **Player XP Bar**                         Never goes backward. Fills with entries, wins, challenges. Tier upgrades broadcast to followers.   Progress feels guaranteed every session --- removes fear of wasted entry

  **Season Progress Bar**                   90-day competitive arc with milestone rewards. Tier promotions auto-share in activity feed.        Long-arc narrative that sustains engagement between individual events

  **Badge Collection**                      Competition, tenure, founding, sponsor, rare event badges. Rarity tiers: Common → Legendary.       Identity investment --- collection psychology drives pursuit of incomplete sets

  **Streak Multipliers**                    7-day login = 1.5x earn. 5-tournament week = 2x. Streak break recovery window.                     Primary frequency driver --- players protect streaks obsessively

  **Token Predictions**                     Spend tokens to call match/tournament outcomes. Winners split pool.                                Twitch Predictions equivalent --- massive live event engagement driver

  **Momentum Surge**                        Triggers when tournament hits 90% capacity --- 2x tokens for remaining entries.                    Hype Train equivalent --- communal urgency that feeds itself

  **Token Leaderboard**                     Weekly top earners visible platform-wide.                                                          Social capital signal --- tokens become identity markers, not just currency

  **[Match Highlight Auto-Share]{.mark}**   [Clip sharing with one tap to Twitter, Discord, TikTok.]{.mark}                                    [Organic platform marketing --- competitive moments drive external discovery]{.mark}
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Community Clip Feed TBD - Discuss with Jonathan]{.mark}

[Need to figure out a clipping system]{.mark}

Momentum surge options

**First Five Surge** --- First 5 players to register earn a bonus token drop. Simple, immediate, rewards the fastest movers. Creates a sprint at registration open instead of a wait-and-see.

**Random Registration Surge** --- The 10th, 25th, or 50th player to register hits a \"lucky spot\" and earns a surprise token bonus. Nobody knows which spot it is until it fires. This keeps every registration feeling like it could be the one.

**Time Window Surge** --- Register within the first 2 hours of a tournament going live and earn bonus tokens. Rewards early movers without being tied to fill rate at all.

**Comeback Surge** --- Fires mid-tournament when a player who lost in round one re-enters a different open tournament within 3 hours. Converts elimination directly into re-engagement.

**Cold Start Surge** --- When a new tournament has zero registrations, the very first player to register earns a disproportionately large token bonus. Solves the empty tournament problem by making being first valuable.

**Sponsor-Triggered Surge** --- A sponsor pays to activate a surprise surge at a random moment during a live event. Players watching the stream or with the app open get notified. Creates appointment viewing and drives opens.

**The design principle shift**

You go from one predictable mechanical trigger to a system that feels alive and responsive. Players stop waiting for 90% and start registering immediately because any registration could be the one that fires a reward. The platform feels like it\'s watching and rewarding them, not just running a fill-rate algorithm.

**The one thing to keep from the original**

The fill-based surge still has a place --- but make it random within a range. Instead of exactly 90%, it fires anywhere between 70-95% with no announcement of the threshold. Players never know if the surge has already fired or is about to. That uncertainty alone changes behavior.

## **Organization Gamification System**

Operational and prestige-driven. Orgs compete for recruits, sponsor deals, and platform standing. Their gamification is a franchise management layer on top of competitive results.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Mechanic**                      **What It Does**                                                                                    **Why It Works For This User**
  --------------------------------- --------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------
  **Org XP Bar + Tier System**      Aggregate XP from all team performance. Starter → Elite Org. Tier badge is the recruiting signal.   Elite Org badge attracts top amateur players. Tier is a commercial credential for sponsors.

  **Org Leaderboard**               Public page ranking orgs by combined performance and token volume.                                  Inter-org rivalry drives platform competition at team level

  **Roster Milestone Badges**       10, 25, 50, 100 active player milestones. Token bonus distributed to all members.                   Growth incentive --- rewards org admins for recruiting and retaining players

  **Org vs Org Rivalry Feed**       Public challenge events with org-level prize pool.                                                  Community event creates organic cross-org competition

  **Internal Token Distribution**   Org admin distributes tokens as performance bonuses from org wallet.                                Management tool and player retention lever --- incentivized from inside the org

  **Sponsor Boost Staking**         Org locks tokens to unlock higher sponsor exposure tier (Phase 2).                                  Token holding becomes a commercial action --- orgs invest in platform standing

  **Public Recruit Page**           Org profile visible to all players --- tier, roster, record, sponsor logos displayed.               Passive recruiting tool --- strong orgs attract players without active outreach
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Tournament Organizer Gamification System**

Reputation-driven and operationally focused. A TO\'s version of a win streak is 10 consecutive full tournaments with zero disputes. Their gamification is built around quality and reliability metrics.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Mechanic**                     **What It Does**                                                                                **Why It Works For This User**
  -------------------------------- ----------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------
  **TO Reputation Score**          Calculated from fill rate, dispute rate, payout speed, player satisfaction. Publicly visible.   Players choose TOs with strong reputations --- reputation is direct revenue driver

  **TO Tier System**               Starter → Community → Verified → Featured → Elite TO. Each tier unlocks platform privileges.    Elite TO gets featured placement, sponsor access, priority support --- commercial incentive to improve

  **Fill Rate Tracker**            Dashboard shows fill % across all events. 85%+ average is the benchmark.                        Operational KPI gamified --- directly tied to AEU rake volume

  **Momentum Surge Integration**   When fill rate spikes + tokens injected + sponsor bonus added, Surge activates publicly.        TOs now have live hype mechanics tied directly to business success

  **Sold Out Badge**               Awarded per tournament filled to 100%.                                                          Per-event prestige --- TOs display this on profile as social proof

  **Reliability Streak Badge**     10 consecutive events: zero disputes, zero cancellations.                                       Operational excellence equivalent of a win streak --- rewards the behaviors AEU needs

  **Featured TO Badge**            Platform-selected based on quality metrics --- not purchasable.                                 Curated prestige --- players trust platform-endorsed TOs
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Operational KPIs]{.mark}

**The ones that directly tie to AEU revenue and reputation:**

**Fill Rate** --- percentage of available tournament slots that were filled. This is the single most important TO metric. A TO consistently running 85%+ fill rates is generating maximum rake and proving they can market their events. Below 60% consistently means something is wrong --- pricing, timing, game choice, or the TO\'s audience.

**No-Show Rate** --- percentage of registered players who checked in versus actually showed up to play. High no-show rates trigger refunds and disputes, both of which cost AEU money and operational time. A TO who drives low no-shows is protecting margin.

**Dispute Rate** --- number of formal disputes filed per tournament as a percentage of total matches played. This is a quality signal. High dispute rates mean bad rules, bad score reporting, or a bad community. It also eats AEU\'s operational bandwidth.

**Time to Settlement** --- how quickly a TO closes a tournament and pays out winners after the final match. Slow settlement destroys player trust and increases chargeback risk.

**Player Satisfaction Score** --- post-tournament rating from players who participated. Direct signal of whether the TO is running quality events or just running events.

**Repeat Registration Rate** --- percentage of players who played in a TO\'s event and registered for another one of their events later. This is the real quality metric. Players vote with their wallet.

**Prize Payout Accuracy** --- were prizes paid correctly and on time. One botched payout can permanently damage a TO\'s reputation on the platform.

## **Sponsor Gamification System**

Performance marketing-driven. Sponsors compete with each other within categories and progress through tiers based on campaign performance and renewal streaks. Their \'leveling up\' is graduating to higher placement access.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Mechanic**                             **What It Does**                                                                                                   **Why It Works For This User**
  ---------------------------------------- ------------------------------------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------------------------
  **Sponsor Tier System**                  Bronze → Silver → Gold → Platinum. Each tier unlocks better placement, audience access, and exclusivity options.   Progression tied to spend and performance --- sponsors who run effective campaigns advance faster

  **Campaign Performance Score**           Each campaign scored on impressions, CTR, token redemptions, and conversions. Score drives tier progress.          Data-backed progression --- ROI directly determines platform standing

  **Category Leaderboard**                 Sponsors ranked within their product category (e.g. energy drinks vs energy drinks).                               Competitive pressure within category --- brands invest more to outrank competitors

  **[Category Exclusivity Lock]{.mark}**   [First sponsor in a category to reach Gold tier locks out same-tier competitors.]{.mark}                           [Scarcity mechanic --- creates urgency for brands to move fast before competitor locks them out]{.mark}

  **Campaign MVP Naming Rights**           Platinum sponsors name the MVP badge in their events --- \'Whataburger MVP\'.                                      Brand embedded permanently in player competitive identity --- highest-value placement

  **Renewal Streak Benefit**               Consecutive campaign renewals unlock loyalty tier benefits --- better rates and priority.                          Long-term sponsor retention incentive --- mirrors subscription tenure logic

  **Launch Partner Badge**                 Beta-window sponsors only --- permanent on platform profile.                                                       Founding sponsor prestige --- rewards early commitment with permanent visibility
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Can we create some H2H competitions between sponsors?]{.mark}

# **The Unified Engagement Loop**

Every action in the ecosystem feeds the next. This is not a feature set --- it is a competitive social economy where player behavior drives sponsor outcomes, sponsor investment drives player engagement, and every transaction generates revenue for AEU at the center.

  ------------------------------------------------------------------------------------------
  **1.** Player enters tournament → earns base tokens regardless of result
  ------------------------------------------------------------------------------------------
  **2.** Org benefits from aggregated team XP --- tier advances

  **3.** TO fill rate increases → reputation score improves → more players discover events

  **4.** Sponsor sees engagement spike → funds token multiplier campaign

  **5.** Momentum Surge activates → 2x tokens for all entries during surge window

  **6.** Tokens distributed to players → circulate back into marketplace

  **7.** Players use tokens in marketplace → sponsor sees GMV lift

  **8.** Sponsor upgrades to higher tier → better placement unlocked

  **9.** TO earns Sponsor-Ready badge → more sponsors approach their events

  **10.** Org gains prestige from combined performance → attracts better players

  **11.** Player rank increases → social feed broadcasts → external discovery

  **12.** New players arrive via social share → loop begins again
  ------------------------------------------------------------------------------------------

## **System Intersections**

Four intentional connection points where one user type\'s behavior directly affects another\'s outcomes.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Intersection**              **How It Works**
  ----------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Player ↔ Sponsor**          Sponsor-funded token drops create earn events for players. Sponsor badge campaigns embed brand identity in player profile permanently. Player predictions and watch-to-earn drive viewership that sponsor overlay inventory depends on.

  **Player ↔ Org**              Org internal token distribution gives admins a management tool that directly affects player motivation. Individual player XP aggregates into org-level standing. Being recruited by an org is itself a status moment for the player.

  **TO ↔ Player**               TO fill bars create urgency that drives player registration. TO early registration token incentives reward players who commit early. Player satisfaction ratings directly build or damage the TO\'s reputation score.

  **TO ↔ Sponsor**              Sponsor-Ready TO badge tells sponsors which TOs are worth working with. Sponsor-funded token prize pools run through TO events --- TO gets bigger prize pool, sponsor gets placement. Featured TO status makes events more attractive to sponsors.

  **Marketplace ↔ All Roles**   Marketplace activates at emotional high-intent moments for every user type --- win credits, event registrations, live streams. Every purchase generates token cashback that re-enters the competitive loop.

  **Token ↔ All Roles**         The token is the connective tissue of the entire ecosystem. Players earn it competing, orgs distribute it internally, TOs use it to boost events, sponsors fund it as engagement fuel. Every token transaction touches AEU\'s economy.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **AEU Unified Identity**                                                                                                                                                                                                                                                                      |
|                                                                                                                                                                                                                                                                                               |
| Competitive progression engine · Social status network · Token-powered incentive economy · Sponsor-funded engagement machine · Marketplace-integrated ecosystem · Operational reputation platform Each user type sees a different dashboard. Underneath: one unified behavioral architecture. |
+===============================================================================================================================================================================================================================================================================================+
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Amateur Esports Union \| Most Vicious Players LLC \| Confidential --- Internal Use Only

**AEU TOKEN STRATEGY**

Dual-Layer Token Architecture --- Native to Web3

Internal Strategy \| Jordan Taylor, CEO & Co-Founder

# The Big Picture

The goal is simple: build a token economy that works for players, orgs, TOs, and sponsors from day one --- without the complexity of blockchain getting in the way of launch or adoption.

We do this in two layers. A native platform token first, exactly like COD Points or V-Bucks. Every gamer already understands this model --- zero education required. Then, when the time is right, we open a Web3 token layer on top of it for the utilities that actually benefit from blockchain: ownership, transferability, staking, and scarcity.

The key is that we build the bridge between them from the start --- not as an afterthought.

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Why this order matters**                                                                                                                                                                                                                                                                                                                             |
|                                                                                                                                                                                                                                                                                                                                                        |
| Most Web3 projects launch the blockchain token first and try to build utility around it after. That\'s backwards. By launching the native token first, we tune the entire economy with real behavioral data before opening the Web3 layer. We know our earn rates, redemption patterns, and what players actually engage with --- instead of guessing. |
+========================================================================================================================================================================================================================================================================================================================================================+
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **LAYER 1**                                                           |
|                                                                       |
| **The Native Token**                                                  |
|                                                                       |
| Platform currency. Controlled environment. Zero blockchain friction.  |
+=======================================================================+
+-----------------------------------------------------------------------+

What It Is

The native token is an internal platform currency --- think COD Points, Fortnite V-Bucks, or League of Legends RP. It lives entirely within AEU. Players earn it by competing. They spend it on entry fees, gear discounts, premium access, and marketplace items. Sponsors fund token drops as part of their campaigns.

It never touches a blockchain. It has no market price. It is purely behavioral and transactional --- which is exactly what we need at launch.

What It Does

+---------------------------------------+--------------------------------------------+
| **Players Earn By**                   | **Players Spend On**                       |
|                                       |                                            |
| > • Winning matches and tournaments   | > • Hybrid tournament entry (USD + tokens) |
| >                                     | >                                          |
| > • Completing performance challenges | > • Premium tournament access              |
| >                                     | >                                          |
| > • Daily check-ins and streaks       | > • Sponsor gear discounts                 |
| >                                     | >                                          |
| > • Referring new players             | > • Token-exclusive marketplace drops      |
| >                                     | >                                          |
| > • Purchasing sponsor products       | > • Ad-free premium stream viewing         |
| >                                     | >                                          |
| > • Keeping winnings in wallet        | > • Micro-tips to players during streams   |
+=======================================+============================================+
+---------------------------------------+--------------------------------------------+

Why It Works for AEU Right Now

> • No regulatory complexity --- native platform currency is standard practice across gaming
>
> • Players already understand the model --- no onboarding friction
>
> • Sponsors can fund token drops immediately as part of their campaigns
>
> • We collect real behavioral data on earn/spend patterns before committing to Web3 economics
>
> • Keeps the competitive engine clean --- tokens enhance the platform, they don\'t complicate it

+-----------------------------------------------------------------------+
| **LAYER 2**                                                           |
|                                                                       |
| **The Web3 Token**                                                    |
|                                                                       |
| Ownership. Transferability. Staking. Scarcity.                        |
+=======================================================================+
+-----------------------------------------------------------------------+

What It Is

The Web3 token is a separate blockchain-based asset that launches after the native token has proven the economy. It is NOT a speculative investment. It is a utility token with real function inside AEU --- but one that benefits from the properties only blockchain can provide: true ownership, transferability between wallets, staking mechanics, and verifiable scarcity.

Not every native token utility makes sense here. We only bring the ones that blockchain genuinely improves.

What Belongs in Web3

+-----------------------------------------------+----------------------------------+
| **Web3 Utilities**                            | **Stays Native Only**            |
|                                               |                                  |
| > • Staking for sponsor placement boosts      | > • Daily login rewards          |
| >                                             | >                                |
| > • Token-gated premium tournament access     | > • Referral bonuses             |
| >                                             | >                                |
| > • Marketplace limited drops (true scarcity) | > • Partial refunds in tokens    |
| >                                             | >                                |
| > • Org stake signal for sponsor matchmaking  | > • Profile completion rewards   |
| >                                             | >                                |
| > • Governance signals (platform voting)      | > • Feedback submission rewards  |
| >                                             | >                                |
| > • Cross-platform transferability            | > • Re-entry bonuses             |
+===============================================+==================================+
+-----------------------------------------------+----------------------------------+

## The Legal Line

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Critical positioning --- must be maintained consistently**                                                                                                                                                                                                                                                                                                                                                            |
|                                                                                                                                                                                                                                                                                                                                                                                                                         |
| The Web3 token must always be positioned as: Utility \| Reward \| Access \| Incentive It must never be positioned as: An investment \| A profit-generating asset \| A speculative instrument This distinction is not just marketing --- it is the difference between a utility token and a security under SEC guidelines. Every document, pitch, and communication about the Web3 token needs to maintain this framing. |
+=========================================================================================================================================================================================================================================================================================================================================================================================================================+
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# The Bridge --- What Has to Be Built Now

This is the most important architectural decision in the entire token strategy. The conversion bridge between the native token and the Web3 token must be designed before either token launches --- not added later.

Here is why it cannot be an afterthought:

> • **The conversion rate between native and Web3 tokens affects how we price and distribute the native token from day one**
>
> • If the smart contract does not include conversion and staking functions at deployment, you cannot add them cleanly later --- you would need a token migration, which is expensive and damages community trust
>
> • Players who accumulate native tokens need to know upfront whether and how those tokens translate to the Web3 layer --- otherwise you create a trust problem at the moment of Web3 launch

## 

## What the Bridge Looks Like

At Web3 launch, native token holders get the option to convert at a defined rate. Example: 1000 native tokens = 1 Web3 token. The rate is set based on actual behavioral data from the native token economy --- how tokens were earned, spent, and circulated. We set it, we announce it early, and we honor it.

This is not complicated to build if it is planned for. It is very complicated to add after the fact.

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **What to tell the dev team tomorrow**                                                                                                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                                                                     |
| When the token architecture conversation comes up --- and it should come up --- the ask is this: design the native token system with a conversion bridge in mind from the start. Even if Web3 is 18-24 months away, the data model and token ledger need to be built in a way that supports eventual migration. This is an architectural requirement, not a future feature request. |
+=====================================================================================================================================================================================================================================================================================================================================================================================+
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# Side-by-Side Comparison

  ---------------------------------------------------------------------------------------------------
                                **NATIVE TOKEN**             **WEB3 TOKEN**
  ----------------------------- ---------------------------- ----------------------------------------
  **Nature**                    Internal platform currency   Blockchain-based utility token

  **Blockchain**                None                         Yes (TBD chain --- Solana, Base, etc.)

  **Market price**              No --- AEU controlled        Yes --- market determined

  **Transferable**              No --- stays in AEU          Yes --- wallet to wallet

  **Staking**                   No                           Yes --- built into smart contract

  **Regulatory risk**           Minimal                      Managed via utility positioning

  **Launch timing**             Beta launch                  Post-product-market fit

  **Player education needed**   Zero                         Low --- convert existing users

  **Primary purpose**           Behavioral economy           Ownership + advanced utility
  ---------------------------------------------------------------------------------------------------

# Recommended Sequencing

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Phase**     **Timing**                **Token Action**
  ------------- ------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------
  Now           Pre-launch                Design native token economy. Define earn rates, spend categories, and conversion bridge architecture. Brief dev team.

  Beta          Spring 2026               Launch native token. Monitor earn/spend behavior. Tune rates. Build behavioral dataset.

  Post-Beta     6-12 months post-launch   Analyze token data. Define Web3 utility shortlist. Engage blockchain architect. Design smart contract with staking and conversion built in.

  Web3 Launch   18-24 months out          Deploy Web3 token. Open conversion bridge at announced rate. Activate staking and advanced utilities.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# The One Risk to Stay Ahead Of

The token economy math. This is the part most founders skip and it creates real problems.

If players earn tokens too easily and redeem them too freely, you have built a discount engine that bleeds margin. If tokens are too hard to earn, nobody engages with the system and the whole economy goes flat. The earn rate and redemption rate need to be modeled before beta launch --- even roughly --- so you have a baseline to measure against.

This is not a blockchain problem. It is an economics problem, and it applies to the native token just as much as the Web3 token. Get a basic token economy model built alongside the product so the team is tuning against real targets from day one.

Amateur Esports Union \| Most Vicious Players LLC \| Confidential --- Internal Use Only


---

# PART 5: Affiliate Program — Membership Conversion Model (v2)

**[AMATEUR ESPORTS UNION]{.smallcaps}**

**Affiliate Program**

*Membership Conversion Model \| v2 Working Document*

February 2026 \| Internal \| Not for Distribution

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **SCOPE DECISION This affiliate program is membership conversion only.**                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                       |
| Payouts trigger exclusively when a referred user converts to a paid Premium membership. All other affiliate triggers from the v1 architecture (rake share, GMV, marketplace, tokens) are deferred. This decision simplifies the model, eliminates the commission stack problem, and makes financial modeling tractable before launch. |
+=======================================================================================================================================================================================================================================================================================================================================+
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# **The Model**

The AEU affiliate program operates on a single trigger with a two-part payout. Every user type --- players, orgs, tournament organizers, and sponsors --- can participate under the same structure.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Element**                        **Structure**                                                           **Notes**
  ---------------------------------- ----------------------------------------------------------------------- --------------------------------------------------------------------------------------------------
  **Trigger**                        Referred user converts to paid Premium membership                       Free signups do not trigger payout. Conversion required.

  **Payout --- Part 1**              One-time % of month one subscription fee                                Paid upon confirmed conversion. Incentivizes the referral act.

  **Payout --- Part 2**              Small recurring % of monthly fee while member stays subscribed          Incentivizes referrer to advocate for retention, not just acquisition. Capped at 12 months.

  **Recurring Cap**                  12 months maximum per referred member                                   Bounds the long-tail liability. Prevents indefinite obligation on a single referral.

  **Who Can Refer**                  All user types --- Players, Orgs, TOs, Sponsors                         Same payout structure applies across all types unless later differentiated

  **What Does NOT Trigger Payout**   Tournament entries, marketplace GMV, token purchases, event fill rate   All other affiliate triggers from v1 doc are deferred. This model is membership conversion only.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# **Payout Structure Illustration**

The actual dollar amounts depend on two decisions still to be made: the one-time conversion percentage \[A%\] and the recurring monthly percentage \[B%\]. The structure below illustrates how the math works at different price points once those percentages are set.

+----------------------------------+-----------------------------+-------------------------------+-------------------+-----------------------+----------------------------+
| **Subscription Price (Monthly)** | **One-Time Payout (TBD %)** | **Monthly Recurring (TBD %)** | **Months Active** | **Max Recurring**     | **Total Max Per Referral** |
|                                  |                             |                               |                   |                       |                            |
|                                  |                             |                               |                   | **Earned**            |                            |
+==================================+=============================+===============================+===================+=======================+============================+
| **\$X / month**                  | \$X × \[A%\]                | \$X × \[B%\]                  | Up to 12          | \$X × \[B%\] × 12     | One-time + Max Recurring   |
+----------------------------------+-----------------------------+-------------------------------+-------------------+-----------------------+----------------------------+
| **Example: \$9.99**              | \$9.99 × \[A%\]             | \$9.99 × \[B%\]               | Up to 12          | \$9.99 × \[B%\] × 12  | Depends on A% and B%       |
+----------------------------------+-----------------------------+-------------------------------+-------------------+-----------------------+----------------------------+
| **Example: \$19.99**             | \$19.99 × \[A%\]            | \$19.99 × \[B%\]              | Up to 12          | \$19.99 × \[B%\] × 12 | Depends on A% and B%       |
+----------------------------------+-----------------------------+-------------------------------+-------------------+-----------------------+----------------------------+

*The 12-month cap means the maximum affiliate obligation per referred member is known and bounded from day one --- which is critical for financial modeling at scale.*

**Scale Impact: What This Costs AEU**

Before percentages are finalized, it\'s worth stress-testing the model at different referral volumes. This is the table Jordan and Maurice need to run once the subscription price and percentages are decided.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Referrals Converted**   **One-Time Payout Total**   **Avg Months Active**   **Recurring Payout Total**   **Total Affiliate Cost to AEU**                     **Note**
  ------------------------- --------------------------- ----------------------- ---------------------------- --------------------------------------------------- ----------
  **10 referrals**          10 × \$X × \[A%\]           6 months avg            10 × \$X × \[B%\] × 6        Manageable at any price point                       Beta

  **100 referrals**         100 × \$X × \[A%\]          8 months avg            100 × \$X × \[B%\] × 8       Needs to be modeled at price point                  Growth

  **1,000 referrals**       1,000 × \$X × \[A%\]        10 months avg           1,000 × \$X × \[B%\] × 10    Must be in financial model before committing to %   Scale
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The 1,000-referral row is the one that needs careful modeling. At scale, the recurring tail across thousands of active referrers could represent a significant monthly obligation --- manageable if built into the financial model, problematic if it isn\'t.

# 

# **Behavior by User Type**

All four user types earn under the same structure, but their referral behavior and volume potential differ significantly. This table is important for forecasting where affiliate growth will actually come from.

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **User Type**              **Referral Method**                                    **Scale Potential**               **Expected Behavior**                                                      **Risk**
  -------------------------- ------------------------------------------------------ --------------------------------- -------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------
  **Player**                 Personal link, social sharing, word of mouth           Low-medium (1--20 referrals)      Organic, authentic --- refers friends from existing gaming circles         Low. Volume naturally capped.

  **Org**                    Org-wide invite link, team recruitment drives          Medium-high (20--200 referrals)   Structured --- orgs recruit players into org AND platform simultaneously   Medium. Large orgs could drive significant recurring obligation.

  **Tournament Organizer**   Event promotion, registration links, stream overlays   High (100--1,000+ referrals)      TOs have large audience reach --- could be your highest volume referrers   High. A successful TO driving 500+ conversions creates real recurring liability. Cap must hold.

  **Sponsor**                Campaign placements, branded content, promo codes      Variable (depends on campaign)    Campaign-driven --- performance tied to activation quality                 Medium. Tied to campaign scope.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

*Tournament Organizers are the highest-risk, highest-reward referral class. A single TO with a large community could drive hundreds of conversions. The 12-month cap is especially important for this group.*

# **Open Questions for Decision**

The model architecture is locked. What remains are six decisions --- most of which require both the subscription pricing and a financial model to answer properly.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **\#**   **Question**                                                              **Why It Matters**                                                                                                                  **Options to Consider**                                                                                                                               **Answer**
  -------- ------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------
  **1**    **What is the one-time conversion percentage \[A%\]?**                    This is the primary referral incentive. Too low and nobody shares. Too high and it erodes month-one revenue.                        *Industry benchmarks: SaaS affiliate programs typically run 20--50% of first month. Gaming platforms often higher (30--50%) to drive early growth.*   30%

  **2**    **What is the recurring monthly percentage \[B%\]?**                      Sets the retention advocacy incentive. Should be meaningfully smaller than A% but still worth caring about.                         *Typical range: 5--15% monthly. Lower than A% by design --- it rewards loyalty, not the acquisition act.*                                             8%

  **3**    **Is the 12-month cap the right ceiling?**                                Too short and it doesn\'t drive long-term retention behavior. Too long and liability compounds at scale.                            *6 months = low liability, weaker incentive. 12 months = strong incentive, manageable math. 24 months = strong but risky pre-scale.*                  yes

  **4**    **Does the % differ by user type?**                                       A player referring a friend is different from a TO driving 500 conversions via event promotion. Should payouts scale differently?   *Option A: Flat rate for all. Simple. Option B: Higher one-time for players (organic), lower recurring for TOs (volume protection).*                  no

  **5**    **What triggers cancellation of recurring payout?**                       If a referred member downgrades to free or cancels --- recurring stops. But what about pauses, refunds, or payment failures?        *Needs clear rules in the payout logic before engineering builds it. Edge cases matter here.*                                                         12 month limit reached - or member cancels

  **6**    **Is there a clawback on the one-time payout for early cancellations?**   If a referred member cancels in month one, does the referrer keep the one-time payout? Most programs say yes, but worth deciding.   *Standard practice: one-time payout is non-refundable after 30-day window. Protects against gaming the system.*                                       NO
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 

# **Next Steps**

## **To Finalize the Model**

-   Stress-test the scale table at 100, 500, and 1,000 conversions once price is set

## **To Build for Program Launch**

-   Unique referral link generation per user (all four types)

-   Conversion tracking --- free signup vs. paid conversion attribution

-   One-time payout trigger on confirmed Premium conversion

-   Recurring payout logic with 12-month counter per referred member

-   Basic affiliate dashboard: referral count, conversion rate, earnings to date

*AEU Internal Working Document \| Confidential \| Not for Distribution*


---

# PART 6: Website Feature Access Matrix (Free vs Gold vs Platinum)

**AEU WEBSITE FEATURE ACCESS MATRIX**

**(Free vs Gold vs Platinum)**

**1. [PLAYER MEMBERSHIP FEATURES]{.underline}**

**[[Monthly - \$9.99]{.underline} [6 month - \$49.99]{.underline}]{.mark}**

**[[Yearly - \$99.99]{.underline}]{.mark}**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------
  **Feature**                                                                       **Free**             **Gold**             **Notes**
  --------------------------------------------------------------------------------- -------------------- -------------------- -----------------------------
  Account creation & profile                                                        ✅                   ✅                   Platform baseline

  Join public tournaments                                                           ✅                   ✅                   Core usage

  Pay standard entry fees                                                           ✅                   ✅                   No discounts

  Free tournament entries (monthly cap) [(Automated tournaments) (tokens)]{.mark}   ❌                   ✅                   Subscription incentive

  Access to member-only tournaments                                                 ❌                   ✅\                  Scarcity + upgrade pressure
                                                                                                         Stakes x1            

  League & seasonal competition eligibility [(AEU automated leagues)]{.mark}        ❌                   ✅                   Retention driver

  Ranked matchmaking access                                                         ❌                   ✅                   Competitive integrity

  Featured match eligibility (non-stream)                                           ❌                   ✅                   Placement / visibility

  Player stats & performance, ranking history [(dashboard)]{.mark}                  ❌                   ✅ (full + export)   Data value

  Leaderboards (view only)                                                          ✅                   ✅                   Public engagement

  [Make a Team]{.mark}                                                              [Join Only]{.mark}   [✅]{.mark}          [One team per game]{.mark}

  Leaderboards (ranked placement eligibility) [(scouting)]{.mark}                   ❌                   ✅                   Prestige gating
  ---------------------------------------------------------------------------------------------------------------------------------------------------------

**2. [TOURNAMENT ORGANIZER (TO) FEATURES]{.underline}**

**[[Gold; Monthly - \$29.99 , 6 Month - \$149.99 Yearly - \$299.99]{.underline}]{.mark}**

**[[Platinum: Monthly - \$97.99 6 Month - \$449.99 Yearly - \$799.99]{.underline}]{.mark}**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Feature**                                                            **Free**      **Gold**      **Platinum**                    **Notes**
  ---------------------------------------------------------------------- ------------- ------------- ------------------------------- --------------------------------
  Host 1 tournament at a time                                            ✅            ❌            ❌                              Load control

  Host multiple tournaments                                              ❌            ✅ 4          ✅ (unlimited)                  Subscription justification

  Charge entry fees                                                      ❌            ✅            ✅                              Revenue enablement

  League hosting (season scheduling)                                     ❌            ✅            ✅                              Long-term engagement

  Standard bracket formats                                               ✅            ✅            ✅                              Universal

  **Automated prize payouts**                                            ✅            ✅            ✅                              Core trust feature

  **Prize Payout Amounts**                                               \<\$500       \<\$2500      Bid for approvals over \$2500   

  Tournament ruleset configuration                                       ✅            ✅            ✅                              Required flexibility

  Branded brackets (logos & colors only)                                 ❌            ✅            ✅                              Sponsor surface

  Staff / moderator assignment                                           [✅]{.mark}   [✅]{.mark}   ✅                              Ops-heavy

  Priority sponsor placement                                             ❌            ❌            ✅                              Premium visibility

  Priority Tournament visibility                                         ❌            ✅            ✅                              Premium visibility

  **Clickable sponsor links per tournament (sponsor logo placements)**   **0**         **2**         **4+**                          **Primary monetization lever**

  Sponsored tournament designation                                       ❌            ✅            ✅                              Sales positioning

  Data Exporting (dashboard access)                                      ❌            ✅            ✅                              Data value
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------

**[3. SPONSOR MEMBERSHIP FEATURES]{.underline}**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Feature**                       **Base Level**                                **Platinum (Enterprise)**            **Notes**
  --------------------------------- --------------------------------------------- ------------------------------------ ----------------------------------------------
  Membership cost                   FREE                                          Custom / Contracted                  Platinum is not self-serve

  Approval required                 ✅                                            ✅                                   Free sponsors must be approved before access

  Inquiry required                  ❌                                            ✅                                   Platinum is invite / review based

  Eligible sponsor type             Individuals, local brands, small businesses   National & global brands             Clear brand segmentation

  Access to sponsor marketplace     ✅                                            ✅                                   Platinum bypasses marketplace

  Where sponsorship can be placed   Anywhere available in marketplace             Anywhere available in marketplace\   Controlled scale
                                                                                  + Custom placements via inquiry      

  Self-serve sponsorship checkout   ✅                                            ✅                                   Prevents misuse at entreprise level

  Custom sponsorship structures     ❌                                            ✅                                   Platinum flexibility

  Brand safety review               ✅ (standard)                                 ✅ (enhanced)                        Risk control

  Dedicated account handling        ❌                                            ✅                                   Required for large brands

  Dashboard                         ✅                                            ✅                                   Track Data
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------

## **Organization Membership Features Matrix (Free vs Gold vs Platinum)[(Can buy TO memberships)]{.mark}**

+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Feature                                       | ## Gold           | ## Platinum                | ## Notes                                            |
+==================================================+===================+============================+=====================================================+
| ## Org account creation & profile                | ## ✅             | ## ✅                      | ## Baseline org presence                            |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Org public page (logo, socials, bio)          | ## ✅             | ## ✅                      | ## Branding core                                    |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Verification badge eligibility                | ## ✅             | ## ✅                      | ## Verification = trust + recruiting signal         |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Featured org placement in discovery           | ## ❌             | ## ✅                      | ## Platinum visibility lever                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Create teams (quantity)                       | ## 4              | ## Unlimited               | ## Buy additional teams individually                |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Roster seats (players managed)                | ## 50             | ## Unlimited               | ## Seat-based scaling lever                         |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Coach/Manager roles                           | ## ✅             | ## ✅                      | ## Multi-role ops begins at Gold                    |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Admin roles (multi-admin)                     | ## Up to 3 admins | ## Unlimited admins        | ## Seat-based admin pricing optional                |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Finance role + wallet permissions             | ## ✅             | ## ✅                      | ## Adds control + governance                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Org wallet (deposit/hold funds)               | ## ✅             | ## ✅                      | ## Baseline for bulk entry + prize routing          |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Budget allocation by team                     | ## ✅             | ## ✅                      | ## Team budgets & spend caps                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Bulk tournament registration                  | ## ✅             | ## ✅                      | ## Bulk entry is paid feature                       |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Bulk invites (email/Telegram roster import)   | ## ✅             | ## ✅                      | ## Improves onboarding speed                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Player Premium seat bundles (org pays)        | ## ✅             | ## ✅                      | ## Upsell driver + retention                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Access to org-only tournaments/leagues        | ## ✅             | ## ✅                      | ## Competitive ecosystem                            |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Org seasonal standings / rankings             | ## ✅ (compete)   | ## ✅ (compete + featured) | ## Free can view, paid can participate/feature      |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Org performance analytics dashboard           | ## ✅ (standard)  | ## ✅ (advanced + export)  | ## Advanced includes deeper filters + trends        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Player scouting visibility tools              | ## ✅             | ## ✅                      | ## Recruiter value                                  |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Data export (CSV/Excel)                       | ## ❌             | ## ✅                      | ## Export reserved for Platinum                     |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Marketplace org storefront                    | ## ✅ (basic)     | ## ✅ (premium)            | ## Premium storefront = more customization          |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Co-branded sponsor placements on org page     | ## ✅             | ## ✅                      | ## Sponsor value surface                            |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Affiliate tracking & org codes                | ## ✅             | ## ✅                      | ## Org as commerce amplifier                        |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Affiliate payouts dashboard                   | ## ✅             | ## ✅                      | ## Shows GMV + commissions                          |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Sponsor marketplace access (org-level offers) | ## ✅             | ## ✅                      | ## Ability to accept/run campaigns                  |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Sponsor campaign tools (org-level)            | ## ✅ (limited)   | ## ✅ (full)               | ## Full includes more campaign types/slots          |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Priority sponsor matchmaking / inquiries      | ## ❌             | ## ✅                      | ## Platinum "concierge" lever                       |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Prize settlement to org wallet                | ## ✅             | ## ✅                      | ## Baseline if org wins                             |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Prize split rules + player agreement workflow | ## ✅             | ## ✅ (advanced templates) | ## Advanced includes reusable templates + approvals |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Dispute priority support                      | ## ❌             | ## ✅                      | ## Premium support lever                            |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## Dedicated account handling                    | ## ❌             | ## ✅                      | ## Similar to Sponsor Platinum model                |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+
| ## API / integrations (future)                   | ## ❌             | ## ✅                      | ## Optional later (Discord, school SIS, etc.)       |
+--------------------------------------------------+-------------------+----------------------------+-----------------------------------------------------+

## 

