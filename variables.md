# Re:Quest — Variable Reference

Every JavaScript variable across the codebase: what it holds, where it's set, and what reads it.

---

## Shared across files

These exist in **every** page file independently (each file has its own copy — there is no shared module).

| Variable | Type | What it holds |
|----------|------|---------------|
| `app` | Firebase App | The initialized Firebase app instance. Every other Firebase service (`auth`, `db`) is created from this. |
| `auth` | Firebase Auth | Authentication service. Used in `onAuthStateChanged`, `signOut`, `signInWithEmailAndPassword`. |
| `db` | Firestore | Database connection. Passed to every `doc()`, `collection()`, `addDoc()`, `onSnapshot()` call. |
| `firebaseConfig` | Object | Project credentials (apiKey, authDomain, projectId, etc.). Read once by `initializeApp()`. |

---

## home.html

### State variables

| Variable | Type | Set by | Read by |
|----------|------|--------|---------|
| `currentUser` | `FirebaseUser \| null` | `onAuthStateChanged` | Everything that needs the user's UID or email. Guards most write operations. |
| `userProfile` | `Object \| null` | `onAuthStateChanged` (initial `getDoc`), `onSnapshot` (profile listener) | `renderNavProfileCard`, `renderHomeHero`, `placeOrder`, `submitSOS`, `renderConfirmPage`. Holds `fullName`, `nickname`, `role`, `photoURL`, `studentId`, `course`, `yearLevel`, `section`, `bio`, `badges`. |
| `cart` | `Array` | `addToCart`, `quickAdd`, `addToCartFromModal`, `changeCartQty` | `renderCartDrawer`, `renderConfirmPage`, `computeFees`, `updateAllCartCounts`, `placeOrder`. Each entry: `{ id, name, emoji, price, qty }`. |
| `selectedStore` | `Object \| null` | `openStoreMenu()` | `renderMenuPage` — determines which store's items to show. Set to a `STORES` entry. |
| `searchQuery` | `string` | Search input on the menu page | `renderMenuPage` — filters `MENU` items by name/description. |
| `storeSearchQuery` | `string` | Search input on the stores list page | `renderStoresPage` — filters the `STORES` array. |
| `avatarUploading` | `boolean` | `handleAvatarFileChange` (set true at start, false in finally) | `renderProfileModal` — swaps the "+" upload button for a spinner. |
| `storeStatuses` | `Object` | `onSnapshot` on the `stores` Firestore collection | `renderStoresPage`, `openStoreMenu` — determines the closed overlay and admin toggle button state. Shape: `{ [storeId]: { closed, closedBy, closedAt } }`. |
| `pendingAcceptId` | `string \| null` | `acceptSOS(id)` | `confirmAcceptSOS` — the SOS post id waiting for the user to confirm acceptance. |
| `currentFulfillment` | `Object \| null` | `confirmAcceptSOS` (copied from `SOS` array) | `renderFulfillmentPage` — holds the full SOS data for the fulfillment page after the post is deleted from Firestore. Cleared by `backToBoard()`. |
| `receiptOrderUnsub` | `Function \| null` | `subscribeToReceiptOrder()` | Called at the start of `subscribeToReceiptOrder()` to cancel the previous listener before setting up a new one. |
| `allOrdersFeed` | `Array` | `subscribeToOrdersFeed()` onSnapshot | `renderMyOrders`, `updateMyOrdersDot`. Holds every order doc (`{ id, userId, status, items, total, riderId, ... }`). Admins see all; students see only their own. |
| `myOrdersFilter` | `string` | `setMyOrdersFilter()` | `renderMyOrders` — one of `'all'`, `'pending'`, `'accepted'`, `'delivered'`. |
| `SOS` | `Array` | `onSnapshot` on `sos_posts` Firestore collection | `renderSOSBoard`, `confirmAcceptSOS`. Each entry: `{ id, item, name, location, contact, fb, duration, reward, itemEmoji, status, postedAt }`. |
| `allUsersCache` | `Array` | `ensureUserSearchLoaded` onSnapshot on `users` collection | `renderUserResults`, `filterUsersDir`. Holds every user profile (excluding current user). |
| `userSearchLoaded` | `boolean` | `ensureUserSearchLoaded` (set true on first call) | Guards `ensureUserSearchLoaded` so it only sets up the Firestore listener once. |
| `toastTimer` | `number \| null` | `showToast` | `clearTimeout(toastTimer)` at start of each `showToast` call — prevents stacked timeouts. |

### Constants

| Constant | Type | Value / purpose |
|----------|------|-----------------|
| `STORES` | `Array` | 21 stall objects: `{ id, name, emoji, desc, color }`. The `id` is the numeric key that links to `MENU` items. |
| `MENU` | `Array` | All menu items: `{ id, storeId, name, price, desc, emoji }`. Filtered by `storeId` when a store is opened. |
| `SOS_FLAT_FEE` | `number` | `20` — the flat ₱20 borrow fee applied to every SOS post. Change here to update all new posts. |

### Key Firestore collections (home.html)

| Collection | Read by | Written by |
|------------|---------|------------|
| `users/{uid}` | `onAuthStateChanged`, `ensureUserSearchLoaded` | `saveNickname`, `saveBio`, `handleAvatarFileChange`, `setDoc` on sign-up (index.html) |
| `orders` | `subscribeToOrdersFeed` | `placeOrder`, `cancelMyOrder` (deletes), `adminDeleteOrder` |
| `sos_posts` | `onSnapshot sosQuery` | `submitSOS` (addDoc), `confirmAcceptSOS` (deleteDoc) |
| `stores/{storeId}` | `onSnapshot storeStatuses listener` | `toggleStoreClosed` (admin only) |

---

## relay.html

### State variables

| Variable | Type | Set by | Read by |
|----------|------|--------|---------|
| `currentUser` | `FirebaseUser \| null` | `onAuthStateChanged` | All auth guards, every Firestore write. |
| `userProfile` | `Object \| null` | `subscribeMyProfile` onSnapshot | `renderMyProfileCard`, `populateAvatar`, `getOrCreateConversation`, `handleAvatarFileChange`. |
| `avatarUploading` | `boolean` | `handleAvatarFileChange` | `avatarWrapHtml` — toggles upload button vs spinner. |
| `activeSection` | `string` | `switchSection()` | `handleInlineSearch` — controls which list (`dmList` or `deliveryList`) shows. Values: `'dm'` or `'delivery'`. |
| `activeConversationId` | `string \| null` | `openConversation()` | `renderDMList` (highlights active card), `subscribeMessages`, `sendMessage`. Cleared by `closeThread()`. |
| `activeDeliveryChatId` | `string \| null` | `openDeliveryChat()` | `renderDeliveryList`, `subscribeDeliveryMessages`, `sendDeliveryMessage`. Cleared when a DM is opened. |
| `conversations` | `Map` | `subscribeConversations` onSnapshot | `renderDMList`, `openConversation`. Keys = conversationId (sorted uid pair), values = Firestore conversation doc data. |
| `deliveryChats` | `Map` | `subscribeMyDeliveryChatsAsCustomer`, `subscribeMyDeliveryChatsAsRunner` | `renderDeliveryList`, `openDeliveryChat`. Keys = orderId. |
| `ordersAsCustomer` | `Map` | `subscribeMyOrdersAsCustomer` | `renderDeliveryList` — looks up order metadata (items, total, status) when rendering delivery chat rows. |
| `ordersAsRunner` | `Map` | `subscribeMyOrdersAsRunner` | `renderDeliveryList` — same, but for orders the signed-in user is running. |
| `dmMessagesUnsub` | `Function \| null` | `subscribeMessages()` | Called at the top of `subscribeMessages` to cancel the previous message listener before opening a new conversation. |
| `deliveryMessagesUnsub` | `Function \| null` | `subscribeDeliveryMessages()` | Same pattern as `dmMessagesUnsub`, for delivery chat threads. |
| `allUsersCache` | `Array` | `ensureUserSearchLoaded` | `renderUserResults`, `handleInlineSearch`, `startConversationWith`. |
| `userSearchLoaded` | `boolean` | `ensureUserSearchLoaded` | Guards the Firestore listener so it only fires once. |
| `toastTimer` | `number \| null` | `showToast` | `clearTimeout` guard in `showToast`. |

### Constants / helpers

| Name | Purpose |
|------|---------|
| `backArrowSvg` | SVG string for the back button in the thread header. Referenced in `openConversation` and `openDeliveryChat`. |

### Key Firestore collections (relay.html)

| Collection | Read by | Written by |
|------------|---------|------------|
| `users/{uid}` | `onAuthStateChanged`, `subscribeMyProfile`, `ensureUserSearchLoaded` | `handleAvatarFileChange` |
| `conversations/{cid}` | `subscribeConversations` | `getOrCreateConversation`, `sendMessage` (updates `lastMessage`, `unreadCount`) |
| `conversations/{cid}/messages` | `subscribeMessages` | `sendMessage` |
| `orders` | `subscribeMyOrdersAsCustomer`, `subscribeMyOrdersAsRunner` | (read-only in relay) |
| `deliveryChats/{orderId}` | `subscribeMyDeliveryChatsAsCustomer/Runner` | Created by runner.html when a runner accepts an order |
| `deliveryChats/{orderId}/messages` | `subscribeDeliveryMessages` | `sendDeliveryMessage` |

---

## runner.html

### State variables

| Variable | Type | Set by | Read by |
|----------|------|--------|---------|
| `currentRunner` | `FirebaseUser \| null` | `onAuthStateChanged` | All Firestore writes, auth guards. Named `currentRunner` (not `currentUser`) to distinguish the runner context. |
| `runnerProfile` | `Object \| null` | `onAuthStateChanged` (getDoc on `users/{uid}`) | `renderDashboard`, `acceptOrder`, sign-up form pre-fill. Holds `fullName`, `phoneNumber`, `role`, `badges`. |
| `orders` | `Array` | `onSnapshot` (set in `subscribeOrders`) | `renderOrders`, `setFilter`. Full array of all orders from Firestore. |
| `ordersUnsub` | `Function \| null` | `subscribeOrders()` | Called in `handleLogout` to tear down the listener on sign-out. |
| `activeFilter` | `string` | `setFilter()` | `renderOrders` — one of `'pending'`, `'mine'`, `'delivered'`, `'all'`. |
| `selectedOrderId` | `string \| null` | `acceptOrder`, `openOrderDetail` | `confirmAccept` — tracks which order the runner is mid-accepting. |

### Key Firestore collections (runner.html)

| Collection | Read by | Written by |
|------------|---------|------------|
| `users/{uid}` | `onAuthStateChanged` | `submitRunnerApplication` (addDoc to `runner_applications`), promotion is done manually in Firestore |
| `orders` | `onSnapshot subscribeOrders` | `acceptOrder` (updateDoc: sets `status`, `riderId`, `riderName`), `markDelivered` (updateDoc: sets `status:'delivered'`) |
| `runner_applications` | — | `submitRunnerApplication` (addDoc) |
| `deliveryChats/{orderId}` | — | `acceptOrder` (setDoc — creates the delivery chat doc when a runner accepts) |

---

## index.html

Handles sign-in and sign-up only. No persistent state beyond the auth flow.

| Variable | Type | Purpose |
|----------|------|---------|
| `currentUser` | `FirebaseUser \| null` | Set in `onAuthStateChanged`. If already signed in, redirects straight to `home.html`. |

Firestore write: `setDoc(users/{uid}, { fullName, email, studentId, course, yearLevel, section, role:'student', createdAt })` — runs once on successful sign-up.

---

## repost.html

Self-contained. Uses Firestore independently for the freedom wall.

| Variable | Type | Purpose |
|----------|------|---------|
| `posts` | `Array` | Live cache of all `repost_posts` docs. Updated by `onSnapshot`. Each entry: `{ id, type, text, imageData, color, authorName, createdAt }`. |
| `currentPage` | `number` | Which page of the bulletin board is showing. Incremented/decremented by the pagination arrows. |
| `POSTS_PER_PAGE` | `number` | Constant — how many posts fit per board page before pagination kicks in. |
| `db` | Firestore | Same Firebase project. Posts expire after 12 hours via a `createdAt` timestamp check on render. |

Firestore collection: `repost_posts` — addDoc on submit, onSnapshot for live feed. Expired posts (older than 12 hrs) are filtered client-side on render, not deleted server-side.

---

## reprint.html

Stateless — no Firebase. Calculates print job cost client-side and lets the user submit a Google Form or contact a runner.

No persistent JS variables. All state lives in the form inputs and is read once on submit.

---

## Cross-file connections

| Data flow | How it works |
|-----------|-------------|
| **Login → home** | `index.html` signs the user up/in via Firebase Auth. `onAuthStateChanged` in `home.html` fires and loads the `users/{uid}` Firestore doc into `userProfile`. |
| **Order placed → runner sees it** | `home.html` `placeOrder()` writes to `orders`. `runner.html` `subscribeOrders()` has an active `onSnapshot` on the same collection — the new doc appears instantly. |
| **Runner accepts → customer sees status** | `runner.html` `acceptOrder()` updates `orders/{id}.status` to `'accepted'`. `home.html` `subscribeToReceiptOrder()` is watching the same doc — the Cancel button disappears automatically. |
| **Runner accepts → delivery chat opens** | `runner.html` `acceptOrder()` also calls `setDoc` on `deliveryChats/{orderId}`. `relay.html` listeners `subscribeMyDeliveryChatsAsCustomer/Runner` pick this up and add the chat to the delivery list. |
| **Avatar upload** | `home.html` or `relay.html` `handleAvatarFileChange()` resizes the image and saves a base64 JPEG to `users/{uid}.photoURL`. Every other page reads this same field when rendering avatars. |
| **Dark mode** | Toggled by `home.html` profile panel. Writes `'dark'` to `localStorage('requestTheme')`. Every page has a blocking IIFE at the top that reads this before first paint. |
| **Badges** | Stored as a `badges: ['founder', 'developer']` array on `users/{uid}`. Read by `getUserBadges()` in `home.html` and `relay.html`. Set manually in Firestore — no UI for it yet. |
