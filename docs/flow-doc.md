# **TubeYou Gym - Scope and Narratives Design Document**

## **1. Introduction**

The TubeYou Gym is a structured, narrative-driven simulation of the core YouTube viewer and lightweight creator experience. It models the surfaces that viewers use most often: home discovery, search, watch pages, Shorts, channel pages, subscriptions, playlists, saved libraries, account settings, reporting, and local browser state verification. The goal is not to reproduce every production YouTube feature, but to provide a focused environment where users can practice realistic navigation, content discovery, playback, engagement, playlist management, reporting, and account preference workflows.

The Gym models YouTube's primary user hierarchy - **Global Shell -> Discovery Surfaces -> Playback Surfaces -> Channel Surfaces -> Personal Library -> Account and Verification Tools**. It provides faithful representations of the common interfaces: a global header with search and account controls, collapsible guide sidebar, responsive video grids, watch page with metadata and actions, comments, recommendations, Shorts viewer, channel profile tabs, subscriptions feed, saved playlists, viewer history, account settings, report history, and localStorage verification pages.

The system is made up of separate parts that work together:

* Videos, Shorts, channels, playlists, comments, and accounts are loaded from local data fixtures.
* Viewer state persists across sessions using localStorage domains for session, preferences, library, history, queue, comments, reports, and notifications.
* Search queries, recent searches, channel-specific searches, and result filters shape discovery surfaces.
* Watch actions update viewer state, including watch history, progress, view counters, likes, dislikes, subscriptions, playlists, and queue membership.
* Reports can be submitted for videos, comments, thumbnails, channels, channel art, and profile pictures, with report history and notification state tracked for verification.
* Account settings manage privacy, notifications, playback, downloads, billing, sharing, and advanced account details.
* Verification tooling compares the seeded localStorage baseline with current browser state and exposes added, removed, and modified records with JSON diffs.

The Sandbox shows the main areas users interact with, including the signed-in homepage, signed-out empty states, search results, watch pages, playlist playback, Shorts, channel profiles, channel search, subscriptions, You library overview, watch history, Watch Later, Liked Videos, playlist library, account settings, report history, and verifier utilities.

## **2. Personas**

### **2.1 Signed-In Viewer**

Uses TubeYou to discover videos, watch content, engage with creators, manage subscriptions, save videos, browse history, and customize playback or notification settings. The signed-in viewer has persistent local state and can use every library, report, playlist, and account surface.

**Example Use Case:** Jane opens the homepage, filters recommendations with topic chips, watches a video, likes it, saves it to Watch Later, comments on it, subscribes to the channel, and later finds the item again from the You page or Watch History.

### **2.2 Signed-Out Visitor**

Can browse public discovery surfaces, search, open watch pages, view channels, and use basic navigation, but sees sign-in prompts when attempting personal library actions such as Watch History, Watch Later, Liked Videos, or the You overview.

**Example Use Case:** A visitor opens TubeYou, searches for "music", watches a public video, opens the creator channel, and then sees a prompt to sign in when navigating to History or saved videos.

### **2.3 Creator / Channel Owner**

Uses the channel studio-like surfaces to review their own channel presentation and uploaded videos. This persona can open Your Channel and Your Videos pages, review shelves, view captions indicators, and navigate through creator-focused entries exposed through the account and sidebar.

**Example Use Case:** Jane opens Your Channel from the account menu, reviews the channel header, opens Your Videos, scans the For You and Videos shelves, and uses video card menus to inspect actions available for uploaded content.

### **2.4 Content Safety Reviewer**

Uses reporting and verification surfaces to confirm that viewer actions create the expected state. This persona checks report submission flows, report history, notification generation, hidden channel controls, and localStorage diffs.

**Example Use Case:** A reviewer reports a channel from the channel profile menu, opens Report History to confirm the report, then opens `/verify-ls` to confirm the viewer reports and notifications domains show the expected added records.

### **2.5 QA / State Verification Operator**

Uses `/localStorage`, `/verify_raw`, and `/verify-ls` to inspect persisted browser state and compare it with the seeded baseline. This persona validates that no changes appear when no action has been taken, and that each user action produces a meaningful diff.

**Example Use Case:** A QA operator resets verifier state, reloads connected tabs, performs a single action such as hiding a channel, then verifies that `/verify-ls` shows only the intended `hidden-channel:<channelId>` mutation.

## **3. TubeYou Gym Interface Surfaces**

The Sandbox contains simplified UI surfaces, each modeling a core part of the YouTube experience:

**1. Global Header**

**Example Use Case:** Jane uses the hamburger button to expand or collapse the guide, types a query in the search box, reviews recent search suggestions, submits the query to `/results`, opens the microphone affordance, checks notifications, opens create actions, and uses the account avatar to switch accounts or sign out.

**2. Guide Sidebar**

**Example Use Case:** Jane navigates between Home, Shorts, Subscriptions, You, History, Playlists, Watch Later, Liked Videos, Downloads, Report History, Your Channel, Shopping, Music, Premium, YouTube Music, and YouTube Kids from the collapsible sidebar. Subscribed channels appear as channel rows when the viewer is signed in.

**3. Home Feed**

**Example Use Case:** Jane opens `/`, sees the signed-in recommended video grid, uses horizontal topic chips to filter the feed, dismisses a sponsored card, and opens a video card menu to save, share, report, or add the video to a queue.

**4. Signed-Out Home**

**Example Use Case:** A visitor opens the homepage while signed out and sees an empty state encouraging them to search or sign in before a personalized feed is shown.

**5. Search Results**

**Example Use Case:** Jane searches for "man", lands on `/results?search_query=man`, reviews videos, channels, and playlists, opens the advanced filter panel, narrows by upload date, type, duration, and sort order, and scrolls through delayed infinite results.

**6. Watch Page**

**Example Use Case:** Jane opens `/watch?v=<id>`, watches the player, reviews title and metadata, expands the description, likes or dislikes the video, shares it, saves it to a playlist, reports it, and browses related recommendations.

**7. Playlist Watch Panel**

**Example Use Case:** Jane opens a watch URL with playlist parameters, sees the playlist panel beside the player, toggles loop or shuffle, selects another item in the list, and closes the playlist panel to return to standalone watch mode.

**8. Video Actions**

**Example Use Case:** Jane clicks Like to update liked videos, clicks Dislike to update disliked videos, opens Share to copy a link, opens Save to playlist to add the video to Watch Later or another playlist, opens Download options, or reports the video from the More menu.

**9. Comments**

**Example Use Case:** Jane reads comments on a watch page, adds a top-level comment, replies to another comment, reports an inappropriate comment, and sees her local comments persisted under the viewer comments domain.

**10. Shorts Viewer**

**Example Use Case:** Jane opens `/shorts/<id>`, moves through vertical short-form items using next and previous controls, likes a Short, subscribes to the channel, opens comments, shares it, opens remix or sound panels, and uses the overflow menu for reporting.

**11. Channel Profile**

**Example Use Case:** Jane opens `/channel/<channelId>`, reviews the avatar, banner, subscriber count, handle, joined date, view count, description, external links, Subscribe button, and tabs for Home, Videos, Shorts, Live, Podcasts, Playlists, and Posts.

**12. Channel Search**

**Example Use Case:** Jane uses the channel search icon to search within a channel's videos, switches between video and channel scope, and sees matching uploads without leaving the channel profile surface.

**13. Channel Reporting and Hidden User Controls**

**Example Use Case:** A reviewer opens the channel menu, chooses Hide user from my channel, Report channel art, Report profile picture, or Report user, completes the relevant confirmation or multi-step report dialog, and verifies that reports and hidden channel ids are persisted.

**14. Subscriptions Feed**

**Example Use Case:** Jane opens `/feed/subscriptions`, sees a row of subscribed channel avatar filters, reviews uploads grouped by Today, Yesterday, This week, This month, and Older, and opens videos from subscribed creators.

**15. You Overview**

**Example Use Case:** Jane opens `/feed/you`, sees her account header, then browses shelves built from viewer state such as History, Playlists, Watch Later, Liked Videos, and saved or personalized rows.

**16. Watch History**

**Example Use Case:** Jane opens `/feed/history`, searches within watched items, filters by All, Videos, Shorts, Podcasts, or Music, removes individual history entries, pauses watch history, or clears all watch history after confirmation.

**17. Watch Later**

**Example Use Case:** Jane opens `/feed/watch-later` or `/playlist?list=WL`, sees an immersive saved playlist header, filters entries by All, Videos, or Shorts, plays an item, and records the action back into watch history.

**18. Liked Videos**

**Example Use Case:** Jane opens `/playlist?list=LL`, sees videos she has liked, plays an item from the list, and uses playlist-style rows to continue watching liked content.

**19. Playlists Library**

**Example Use Case:** Jane opens `/feed/playlists`, sorts local playlists by recently added or A-Z, reviews playlist cards, and opens a playlist in the `/playlist?list=<id>` route.

**20. Public and Local Playlist Pages**

**Example Use Case:** Jane opens `/playlist?list=<id>` for Watch Later, Liked Videos, local playlists, or fixture playlists. The playlist router selects the correct page content based on the list id and renders rows, empty states, or public playlist details.

**21. Queue Dock**

**Example Use Case:** Jane adds videos to the queue from video cards or overflow menus, opens the queue dock while browsing or watching, removes queued items, clears the queue, and continues playback from a selected queued item.

**22. Account Menu**

**Example Use Case:** Jane opens the avatar menu, sees current account information, switches accounts, signs out, navigates to account pages, opens Your Channel, reviews purchases, opens settings, or selects appearance, location, language, keyboard shortcuts, and Restricted Mode entries.

**23. Account Settings**

**Example Use Case:** Jane opens `/account` and related settings routes to manage account identity links, channel links, privacy toggles, notification preferences, playback settings, download quality, billing verification, connected apps, and advanced user information.

**24. Notifications**

**Example Use Case:** Jane opens the bell menu in the header, reviews generated viewer notifications such as report confirmations, sees unread counts, and marks notifications read when viewed.

**25. Report History**

**Example Use Case:** Jane opens `/reporthistory`, filters by Past month, Past year, or All, reviews grouped video reports with thumbnails and latest report details, expands all reports for a video, and navigates back to the reported video.

**26. Explore and Product Hubs**

**Example Use Case:** Jane opens product and discovery surfaces such as Music, Shopping, Premium, YouTube Music, YouTube Kids, and Downloads. These pages provide focused destination layouts or coming-soon style entries for secondary YouTube experiences.

**27. Creator Channel Surfaces**

**Example Use Case:** Jane opens `/your-channel` or `/your-videos`, sees the channel header and creator shelves, scans uploaded videos, sees captions indicators, and opens uploaded content from creator-oriented cards.

**28. LocalStorage Export**

**Example Use Case:** A QA operator opens `/localStorage`, automatically downloads `localStorage.json`, and uses the exported data to inspect persisted TubeYou state.

**29. LocalStorage Verifier**

**Example Use Case:** A QA operator opens `/verify-ls`, reviews all known localStorage keys, selects a changed key, opens JSON diffs in split or unified mode, resets individual keys or all state to seed, and confirms reset broadcasts reload connected tabs.

**30. Scope and Verification Notes**

**Example Use Case:** A reviewer opens `/scope` to compare implemented features against scope expectations or `/verify_raw` to follow legacy verification links that now point to `/verify-ls`.

## **4. Page Narratives**

### **4.1 Global Shell**

The global shell combines the header and guide sidebar. The header contains the TubeYou logo, guide toggle, search bar, microphone affordance, create menu, notifications menu, settings menu, and account menu. Search suggestions combine title suggestions and recent searches, with recent entries removable from the suggestion list. The sidebar can be expanded or collapsed and highlights the active route. It provides top-level navigation to discovery, personal library, subscriptions, creator, and product hub pages. Signed-in state changes what appears in the account area and personal navigation sections.

### **4.2 Home Feed**

The homepage is the primary discovery surface. Signed-in users see filter chips and a grid of recommended video cards loaded through the infinite videos hook. Video cards expose thumbnails, metadata, hover actions, and overflow actions. A sponsored card can be dismissed or restored. The page uses skeletons while hydration completes so persisted viewer state does not visibly mismatch between server and client rendering. Signed-out users see a centered empty state prompting them to search and watch videos before personalized recommendations can be built.

### **4.3 Search Results**

The results page reads `search_query` from the URL and ranks videos, channels, and playlists using local data. It supports top chips such as All, Shorts, Unwatched, Watched, Videos, Recently uploaded, and Live. The advanced filter panel models YouTube's filters for upload date, type, duration, and sort order. Results include channel rows with Subscribe controls, playlist rows, video rows with thumbnail actions, and save/share/report affordances. Watched state and subscriptions are read from the viewer store so search results reflect personal history.

### **4.4 Watch Page**

The watch page is the main playback experience. It renders the video player, title, metadata, channel details, action row, expandable description, comment section, related video list, playlist panel when list parameters are present, and queue dock sidebar. On load, playback-related effects collapse the sidebar for focus and update viewer history/progress through the viewer store. The action row updates likes, dislikes, favorites, Watch Later, playlist membership, report history, and share state. Related recommendations allow continued browsing without returning to the homepage.

### **4.5 Video Reporting**

Video reporting is available from the watch page action menu and from thumbnail overflow menus. The report dialog collects the reason and optional details, creates a report record in `reportHistory`, and creates a viewer notification when the viewer is signed in. Thumbnail reporting uses a dedicated dialog and stores a safety action connected to the target video. These report mutations are visible in Report History and in the `/verify-ls` Viewer Reports and Viewer Notifications entries.

### **4.6 Comments and Comment Reporting**

The comments section displays fixture comments and local viewer comments. Users can add new top-level comments and replies. Comment report dialogs are available for individual comments and record the target comment id, parent video id, report reason, timestamp, and status. Local comments are persisted under the comments storage domain, while comment reports use the unified reports domain.

### **4.7 Shorts**

The Shorts page presents one short-form item at a time in a vertical media layout. The viewer can navigate forward and backward, like or dislike the Short, subscribe to the channel, open comments, add local comments, share, remix, inspect sound details, or report the item. The Shorts page uses compact panels for comments, share, remix, sound, and menu actions. Engagement actions update the same viewer state domains used by regular videos so likes, subscriptions, and watch history are consistent across formats.

### **4.8 Channel Profile**

The channel page displays profile metadata, banner, avatar, verification state, handle, subscribers, video count, joined date, total views, description, and external links. Tabs organize channel content into Home, Videos, Shorts, Live, Podcasts, Playlists, and Posts. The Subscribe button writes to the viewer library domain and updates subscribed-channel surfaces. Channel search is integrated into the profile and stores per-channel search state so search terms and scope persist for that channel.

### **4.9 Channel Safety Actions**

Channel safety actions live behind the channel profile overflow menu. Hide user from my channel toggles a channel id in `hiddenChannelIds`. Report channel opens a multi-step dialog that gathers a reason, optional details, and selected videos when applicable. Report channel art and report profile picture use confirmation-style dialogs. These actions feed the unified reports storage domain, and hidden-user changes are represented in verifier output as synthetic `hidden-channel:<channelId>` records.

### **4.10 Subscriptions**

The Subscriptions page requires signed-in state. It builds subscribed channels from the viewer library domain, shows horizontal channel avatar pills, and renders videos from subscribed channels grouped by recency. If the viewer has no signed-in session, the page shows a prompt explaining that subscription updates require sign-in. This surface validates the relationship between subscription actions on channel/watch pages and downstream subscription feed content.

### **4.11 You Overview**

The You page is the viewer's personal library overview. It uses the active account to render a channel-style header and shelves derived from current viewer state. Rows can include History, Watch Later, Liked Videos, playlists, and other saved or personalized content. The same card components used in creator and library contexts appear here, but their menus adapt to the shelf type.

### **4.12 Watch History**

Watch History resolves stored watch entries into videos and Shorts, groups them by time labels such as Moments, Today, Yesterday, Last week, or date, and supports search and filtering by media type. Users can remove individual items, pause watch history, or clear all history with confirmation dialogs. The page also reflects account identity and history pause state, making it a primary validation surface for watch and playlist playback actions.

### **4.13 Watch Later and Liked Videos**

Watch Later and Liked Videos use immersive playlist layouts. A sticky header summarizes the playlist name, owner, item counts, privacy state, and play actions. The content area supports All, Videos, and Shorts chips. Empty states explain what will appear once the user saves or likes items. Playing an item records watch history, keeping the library surfaces connected to playback state.

### **4.14 Playlists**

The Playlists library lists locally created and persisted playlists. Users can sort by recent updates or alphabetical title, browse category chips, and open playlist cards. The `/playlist` route delegates to Watch Later, Liked Videos, public fixture playlists, or local playlists based on the `list` query parameter. Save-to-playlist dialogs update the local playlist storage and notify subscribed playlist views through storage events.

### **4.15 Queue**

The queue lets users build a temporary playback list from video cards and overflow menus. Queue state persists in the viewer queue domain and is rendered through the queue dock while the viewer continues browsing or watching. Users can remove individual entries or clear the queue. Queue mutations are visible in `/verify-ls` under Viewer Queue.

### **4.16 Account Dropdown**

The account dropdown is the signed-in command center. It shows the active account, account switcher, sign-out action, links to Google-account-like settings, Your Channel, purchases, data, appearance, language, Restricted Mode, location, keyboard shortcuts, help, and feedback. Switching accounts updates the active account id and changes all account-specific library state. Signing out changes route behavior for personal pages and switches protected surfaces to signed-out prompts.

### **4.17 Account Settings**

The settings area uses a shared settings shell and section components. `/account` covers identity and channel links. Privacy settings manage saved playlist visibility and subscription visibility. Notifications settings cover push preferences, email preferences, and email language. Playback settings manage inline playback, captions, AV1 preference, info cards, stable volume, and ambient mode-like controls. Downloads, billing, sharing, and advanced settings cover device download quality, quick purchase verification, connected apps, and user/channel identifiers.

### **4.18 Notifications**

Notifications are exposed through the header bell and are backed by the viewer notifications storage domain. Report actions create confirmation-style notifications. The notification dropdown shows recent messages and read state, and account settings expose notification preference controls. The verifier treats notifications as a first-class state domain so reset and report flows can validate that notification records are created and cleared correctly.

### **4.19 Report History**

Report History gives viewers a dedicated page for submitted reports. It groups video reports by video, resolves thumbnails and channel names, shows the latest report for each group, and filters by Past month, Past year, or All. This page is scoped to video reports for viewer readability, while the verifier exposes the full underlying report domain including video, comment, channel, and hidden-channel records.

### **4.20 Creator Channel and Your Videos**

Creator-oriented surfaces share `ChannelStudioView`. The channel header is generated from page configuration and active account data. Shelves render creator videos in large and small card formats, including captions indicators where configured. `/your-channel` focuses on the signed-in channel presentation, while `/your-videos` and related rewrites reuse the same layout for uploaded video browsing.

### **4.21 Product Hubs**

Music, YouTube Music, Kids, Premium, Shopping, Downloads, and related destination pages model secondary YouTube entrypoints. Some are lightweight destination pages and some use coming-soon interactions for actions that are out of current scope. They remain visible in navigation to preserve the broader YouTube information architecture while keeping the Gym focused on core viewer workflows.

### **4.22 LocalStorage Verifier**

The verifier compares seeded storage values with current localStorage values. It includes known domains for auth, UI, channel search, viewer session, preferences, library, history, queue, comments, reports, notifications, settings, and recent searches. The All Data view summarizes changed and clean keys. Selecting a key shows record-level diffs, and clicking a changed row opens a JSON diff modal with split and unified modes. Reset to Seed can reset a single domain, while Reset All clears all same-origin localStorage and broadcasts a reset so connected tabs reload from the clean slate.

### **4.23 LocalStorage Export and Legacy Verification**

The `/localStorage` page downloads a one-line `localStorage.json` snapshot of all browser localStorage entries for the current origin. `/verify_raw` remains as a compatibility page for older verification links and directs users to `/verify-ls`. These pages support QA workflows where the operator needs to inspect raw persisted data or compare it against the interactive verifier.

### **4.24 Scope Page**

The scope page and `pg/scope.csv` define the feature inventory used for planning and validation. Scope entries cover navigation, home discovery, search, Shorts, watch experience, comments, channels, subscriptions, library, playlists, account, notifications, personalization, live, creator basics, and later-stage moderation or community features. This document translates that inventory into narrative product flows.

### **4.25 State Model**

TubeYou's browser state is intentionally split into domains so workflows can be validated independently. Session stores signed-in identity and accounts. Preferences store playback, locale, theme, and restricted mode choices. Library stores likes, dislikes, subscriptions, saved videos, favorites, and playlists. History stores watched videos, progress, view counters, and pause state. Queue stores queued videos. Comments stores local comments and replies. Reports stores submitted reports and hidden channel ids. Notifications stores viewer notifications. Settings stores account settings pages. This domain model allows `/verify-ls` to show meaningful changes for each action without comparing a single opaque blob.

Each page narrative focuses on the user-visible behavior, the route where the behavior appears, and the persisted state that connects the flow to the rest of the Gym.
