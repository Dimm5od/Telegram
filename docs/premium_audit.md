# Premium audit (client-side checks vs server-side restrictions)

## Method
- Reviewed all direct checks of premium status via `UserConfig.isPremium()` and server-side premium errors in Android client code.
- Key commands used:
  - `rg -n "isPremium\(" TMessagesProj/src/main/java/org/telegram -S`
  - `rg -n '"[A-Z0-9_]*PREMIUM[A-Z0-9_]*"' TMessagesProj/src/main/java/org/telegram -S`

## 1) What can be activated locally (client-only gate in code)

These are features where client UX is directly unlocked by local premium checks; if premium flag is forced locally, UI/flow unlocks immediately on the client side.

1. **Dialog filters / folders locking and premium limits in UI logic**
   - Non-premium filters are locked in `lockFiltersInternal`.
   - Premium/default values are switched for caption/about/reactions limits via `isPremium()`.

2. **Large file picker pre-check (2GB vs 4GB)**
   - Document picker blocks oversized files based on local premium state.
   - Shared helper `FileLoader.checkUploadFileSize()` also branches only on local `isPremium()`.

3. **Animated emoji usage checks in composer**
   - `checkPremiumAnimatedEmoji(...)` checks local premium and applies restrictions locally for outgoing message composition.

4. **Premium app icons (launcher icons)**
   - Icon selection is blocked/showing paywall based on local `UserConfig.hasPremiumOnAccounts()`.

5. **Some privacy UI options are locked by client check before request**
   - Voice/message privacy options open premium paywall in UI when not premium.

> Note: local unlock means *client-side gate is removable*. This does **not** guarantee backend acceptance for every action (see server section).

## 2) What is clearly restricted by API server (backend-enforced)

These restrictions are confirmed by explicit server errors / server responses handled by client:

1. **Messaging/contact privacy premium restriction**
   - Server may return `PRIVACY_PREMIUM_REQUIRED` when sending messages or inviting users.

2. **Stories posting/search/report premium restriction**
   - Server may return `PREMIUM_ACCOUNT_REQUIRED` (stories/search/report flows handle it explicitly).

3. **Premium gift code/business logic restriction**
   - Server may return `PREMIUM_SUB_ACTIVE_UNTIL_...` when applying premium gift code.

4. **Story posting limits are server-driven**
   - Errors `STORY_SEND_FLOOD_WEEKLY_*`, `STORY_SEND_FLOOD_MONTHLY_*`, `STORIES_TOO_MUCH` come from backend and are cached client-side.

## Key source locations
- Premium state source: `UserConfig.isPremium()`.
- Local gates and limits: `MessagesController`, `FileLoader`, `ChatAttachAlertDocumentLayout`, `ChatActivityEnterView`, `AppIconsSelectorCell`, `PrivacyControlActivity`.
- Backend restrictions: `MessagesController`, `AlertsCreator`, `StoriesController`, `PostsSearchContainer`, `ReportBottomSheet`, `BoostDialogs`.
