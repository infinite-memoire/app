# Phase 2 Open Questions for Bulletproof Planning

## Payment Integration Questions

### **Payment Provider & Strategy**
1. **Which payment provider should we integrate?**
   - Stripe (recommended, most flexible)
   - Apple Pay/Google Pay only (simpler, platform-specific)
   - Square (alternative with good Flutter support)
   - Multiple providers (complex but maximum compatibility)
    > research using gocardless. If possible, use it. Otherwise use stripe.

**RESEARCH ANALYSIS: GoCardless vs Stripe**

**GoCardless Analysis:**
- **Specialized for direct debit payments** (bank account transfers)
- **Lower fees**: ~1% + €0.20 vs Stripe's 2.9% + $0.30
- **Geographic limitations**: Europe, Australia, New Zealand only
- **NO dedicated Flutter SDK** - requires custom platform channels and significant development effort
- **Limited payment methods**: Direct debit only, no credit cards or mobile payments

**Stripe Analysis:**
- **Native Flutter SDK** (`flutter_stripe: ^10.1.1`) with comprehensive documentation
- **Multiple payment methods**: Credit cards, Apple Pay, Google Pay, bank transfers
- **Global coverage**: 135 currencies, 25+ countries
- **Firebase integration**: Seamless webhook processing with Firebase Functions
- **Higher fees but better user experience and conversion rates**

**RECOMMENDATION: Use Stripe**
- Development time savings with native Flutter SDK
- Better user experience with multiple payment methods  
- Required Firebase Functions integration is well-documented
- Global reach for international users

>>>use stripe then

2. **What is the payment model for the app?**
   - One-time purchase fee (amount?)
   - Monthly subscription (price?)
   - Freemium with premium features (what features are premium?)
   - Pay-per-recording model
   - Free with ads (requires different implementation)
   > the user can record for free, but generating the book based on the recordings will require a subscription

3. **When should payment be required?**
   - Before any app usage (immediately after registration)
   - After trial period (how many days/recordings?)
   - Before specific features (which features?)
   - Flexible user choice
   > before running the analysis of the recordings and generation of the book

4. **What happens to existing user data if payment fails or is cancelled?**
   - Delete all data immediately
   - Grace period (how long?)
   - Read-only access to existing recordings
   > read only access
   - Export option before deletion

5. **Should we support payment method management?**
   - Update credit cards
   > yes
   - Multiple payment methods
   > yes
   - Payment history viewing
   > yes
   - Refund request capability
   > no

### **Payment Flow Design**
6. **Where should the payment screen appear in navigation?**
   - Modal overlay that blocks all app usage
   > yes
   - Dedicated payment route in main navigation
   - Integrated into settings screen
   - In-app purchase style popup

7. **Should we implement payment webhooks for server-side verification?**
   - Yes, implement backend payment verification
   - No, rely on client-side payment confirmation only
   > this should be fine, as there is no server
   - Hybrid approach with local fallback

**RESEARCH CLARIFICATION: Backend Server Requirements**

**"No Server" Architecture Explanation:**
Your current Memoire app uses **Firebase as Backend-as-a-Service (BaaS)**, which IS a backend but managed/serverless:
- Firebase Firestore = Database server
- Firebase Functions = Serverless compute  
- Firebase Auth = Authentication server
- Firebase Storage = File storage server

**Payment Webhooks ARE Required for Security:**
Even with "no custom server," payment processing requires server-side verification for PCI compliance:

```
Flutter App → Stripe SDK → Stripe Servers → Firebase Functions (webhook) → Firestore
```

**Required Implementation:**
- Firebase Functions to handle Stripe webhooks
- Server-side signature verification (mandatory for security)
- Subscription lifecycle management

**RECOMMENDATION: Implement Firebase Functions for webhooks**
- Essential for secure payment processing
- Firebase Functions = serverless, no server management
- Well-documented Stripe + Firebase integration patterns

>>> ok, do that

## Settings Screen Questions

### **User Profile Management**
8. **What user profile information should be editable?**
   - Basic info: Name, email, profile picture
   > yes
   - Extended info: Bio, preferences, notification settings
   > bio, age, places the user lived
   - Account info: Password change, account deletion
   > yes
   - Recording preferences: Quality, format, auto-upload settings
   > no

9. **Should users be able to change their authentication method?**
   - Switch from email/password to Google Sign-In
   > no
   - Link multiple authentication methods
   > no
   - Remove authentication methods
   > no
   - Account merging for existing data
   > no

10. **How should profile picture management work?**
    - Device camera integration
    > yes
    - Photo library selection
    > yes
    - Avatar generation service
    > no
    - Optional (default to initials/generic icon)
    > no

### **App Preferences & Settings**
11. **What audio quality options should we provide?**
    - Recording quality: Low/Medium/High (specific bitrates?)
    - Playback quality settings
    - Compression options for upload
    - Storage optimization settings
    > no options, just recording

12. **What notification preferences should be available?**
    - Recording reminders
    > yes, if inactive for 3 days
    - Upload completion notifications
    > yes
    - Account/payment notifications
    > no
    - Marketing communications opt-in/out
    > no

13. **Should we implement app themes/appearance options?**
    - Light/Dark mode toggle
    > no
    - Custom color schemes
    > no
    - Font size options
    > yes
    - Accessibility features
    > yes

### **Data Management Settings**
14. **What data export/backup options should we provide?**
    - Export all recordings to device
    > no
    - Cloud backup to user's cloud storage
    > no
    - Data transfer to another account
    > no
    - Complete account data download (GDPR compliance)
    > yes

15. **How should users manage their storage and data usage?**
    - Storage usage analytics
    > no
    - Automatic cleanup of old recordings
    > clean up 7 days after upload
    - Manual deletion with confirmation
    > yes
    - Storage upgrade options
    > no

## Offline Support & Upload Progress Questions

### **Offline Capability Scope**
16. **What level of offline functionality should we support?**
    - Full offline mode: Record, edit, organize chapters
    - Limited offline: View existing recordings only
    - Recording-only offline: Create new recordings, sync later
    > this one
    - No offline: Require internet for all operations

17. **How should we handle offline recording storage?**
    - Local SQLite database for metadata
    > no
    - Device file system for audio files
    > this
    - Encrypted local storage for security
    > no
    - Automatic cleanup of uploaded files
    > yes

18. **What happens to offline recordings when storage is full?**
    - Alert user and stop recording
    > yes
    - Automatically delete oldest offline recordings
    > no
    - Compress recordings to save space
    > no
    - User choice with storage management UI
    > after notification and stopping recording, allow the user to manage storage by deleting old recordings

### **Synchronization Strategy**
19. **How should we handle sync conflicts?**
    - Server version always wins
    - Client version always wins
    - Timestamp-based resolution
    > this, in princinple
    - User choice with conflict resolution UI
    - Create duplicate recordings for conflicts
    > Explain, what there server in question is? This solution is suposed to have no backend server. Is one necessary?

**RESEARCH CLARIFICATION: Sync Conflicts with Firebase**

**"Server" Definition in This Context:**
The "server" refers to **Firebase Firestore**, which acts as your cloud database server. Your architecture is:

```
Local Device Storage ↔ Sync Logic ↔ Firebase Firestore (the "server")
```

**Conflict Scenarios:**
1. User records offline on Device A
2. User records offline on Device B  
3. Both devices come online and try to sync
4. Firebase Firestore has conflicting timestamps/data

**Timestamp-based Resolution Implementation:**
```dart
// Use Firestore server timestamps for conflict resolution
final serverTimestamp = FieldValue.serverTimestamp();

// When syncing, compare:
if (localRecording.updatedAt > firestoreRecording.updatedAt) {
  // Local version is newer, upload to Firestore
} else {
  // Server version is newer, update local storage
}
```

**RECOMMENDATION: Implement timestamp-based resolution with Firestore server timestamps**
- Firestore provides server-side timestamps that are timezone-independent
- Simple and reliable conflict resolution
- Works well with offline-first architecture

>>> makes sense, do that

20. **When should automatic sync occur?**
    - App launch/resume
    - Wi-Fi connection available
    - Background sync (requires background processing)
    > yes
    - User-initiated sync only
    - Smart sync based on battery/network conditions
    > yes

21. **How should we handle failed uploads?**
    - Automatic retry with exponential backoff (existing)
    > thats fine
    - User notification with manual retry option
    - Queue uploads for later retry
    - Option to cancel failed uploads

### **Upload Progress & User Experience**
22. **How detailed should upload progress indicators be?**
    - Simple progress bar
    > thats enough
    - Detailed: Speed, time remaining, file size
    - Batch upload progress for multiple files
    - Background upload with notifications
    > yes, if possible

23. **Should users be able to use the app during uploads?**
    - Full app functionality during background uploads
    > yes
    - Limited functionality during uploads
    - Block certain operations during uploads
    - Queue operations until uploads complete

## Technical Architecture Questions

### **Local Storage Implementation**
24. **How should we structure local database schema?**
    - SQLite tables mirroring Firebase structure
    - Simplified schema for offline-only data
    - NoSQL document storage approach
    - Hybrid approach with critical data in SQLite
    > is a local db needed? Avoid, if possible

**RESEARCH ANALYSIS: Local Database Requirements**

**Current Riverpod + Firebase Architecture:**
Your app already has excellent state management with Riverpod. For offline recording storage, you can avoid SQLite by using:

1. **File System Storage**: Store audio files directly on device
2. **SharedPreferences**: Store simple metadata (recording paths, timestamps)
3. **Riverpod Providers**: Handle offline/online state switching

**Recommended Offline Approach (No SQLite Needed):**
```dart
// Use simple file-based storage
final offlineRecordingsProvider = FutureProvider<List<Recording>>((ref) async {
  final directory = await getApplicationDocumentsDirectory();
  final files = directory.listSync()
      .where((file) => file.path.endsWith('.wav'))
      .map((file) => Recording.fromFile(file))
      .toList();
  return files;
});
```

**RECOMMENDATION: Avoid SQLite complexity**
- Use device file system + SharedPreferences for metadata
- Leverage existing Riverpod architecture for state management
- Simpler implementation and maintenance

>>> thats reasonable, do that

25. **How should we handle database migrations for local storage?**
    - Automatic migration on app updates
    > yes
    - User confirmation for major schema changes
    - Backup/restore mechanism for failed migrations
    - Cloud migration assistance

### **State Management Integration**
26. **How should offline/online state be managed in Riverpod?**
    - Global connectivity provider
    - Feature-specific offline state
    - Automatic switching between local/remote data sources
    - Manual mode selection for users
    > What is riverpod? WHy is this relevant?

**RESEARCH EXPLANATION: Riverpod State Management**

**What is Riverpod:**
Riverpod is the state management framework your Memoire app already uses extensively. It's the evolution of Provider with compile-time safety and advanced async handling.

**Current Usage in Memoire:**
- `authProvider` - Handles authentication state
- `audioRecordingProvider` - Manages recording state
- Sealed classes for type-safe state representation
- Code generation with `riverpod_annotation`

**Why Relevant for Offline/Online:**
Riverpod excels at managing complex async state and data source switching:

```dart
// Global connectivity provider
final connectivityProvider = StreamProvider<bool>((ref) {
  return Connectivity().onConnectivityChanged
    .map((result) => result != ConnectivityResult.none);
});

// Smart data provider that switches sources
final recordingsProvider = FutureProvider.family<List<Recording>, String>((ref, storyId) async {
  final isOnline = ref.watch(connectivityProvider).value ?? false;
  
  if (isOnline) {
    // Fetch from Firebase and cache locally
    return await FirestoreService.getStoryRecordings(userId, storyId);
  } else {
    // Return cached/offline data
    return await LocalStorageService.getCachedRecordings(storyId);
  }
});
```

**RECOMMENDATION: Use Global Connectivity Provider**
- Automatic switching between local/remote data sources
- Leverages existing Riverpod architecture
- No BuildContext dependency for background operations

>>> ok, do that

27. **Should we implement optimistic updates?**
    - UI updates immediately with local changes
    - Rollback mechanism for failed sync
    - Conflict indicators in UI
    - User notification of pending sync
    > implement the standard approach. Do not overcomplicate it.

### **Performance & Storage Optimization**
28. **How should we manage local storage size?**
    - Automatic cleanup of old/uploaded recordings
    > yes
    - User-configurable storage limits
    > no
    - Storage analytics and recommendations
    > no
    - Warning system for low device storage
    > yes

29. **Should we implement audio compression for local storage?**
    - Real-time compression during recording
    > no
    - Background compression after recording
    > yes
    - User choice of compression levels
    > no
    - Quality preservation for important recordings
    > no

## User Experience & Flow Questions

### **Onboarding for New Features**
30. **How should we introduce Phase 2 features to existing users?**
> the user will not see different phases. The phases are just for development.
    - In-app tutorial/walkthrough
    > yes
    - Feature announcement notifications
    > no
    - Gradual feature rollout
    > no
    - Optional feature adoption
    > no

31. **Should payment and settings setup be part of initial onboarding?**
    - Integrated into registration flow
    > no
    - Separate post-registration setup
    > yes
    - Optional with reminders
    - Mandatory before app usage

### **Error Handling & User Communication**
32. **How should we communicate payment errors to users?**
    - Detailed error messages with solutions
    - Contact support integration
    > yes
    - Alternative payment method suggestions
    - Graceful degradation options

33. **How should we handle and communicate sync/offline errors?**
    - Silent retry with background sync
    > ytes
    - User notification with manual actions
    > no
    - Detailed error logs for support
    > yes
    - Offline mode continuation options
    > no

---

## Priority Questions for Immediate Planning

**CRITICAL (Must answer before implementation):**
- Questions 1, 2, 3, 8, 16, 19, 24, 26 (Core functionality decisions)

**IMPORTANT (Affects implementation approach):**
- Questions 6, 11, 17, 20, 22, 28 (Implementation strategy)

**NICE TO HAVE (Can be decided during implementation):**
- Questions 13, 18, 25, 30 (Enhancement features)

Please provide detailed answers to help create bulletproof Phase 2 implementation plans.