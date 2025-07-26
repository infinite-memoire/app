# Memoire Audio Recording App - Development Plan

## Project Overview

Memoire is an application designed for audio recording, organization, and management. 

The prupose of the app is to enable the user to create a book of 100-250 pages based on audio recordings. Use cases are recording memoires, recording family history, recording the history of an organization.

The mobile app and web frontend allows users to 
- record audio clips, 
- replaying audio clips
- optionally organize them into chapters with topics/themes, 
- store them securely in the cloud.
- start the AI-powered processing
- respond to generated follow up questions

The web frontend allow the user to do all the things, the mobile app does, plus  
- user settings, billing, invoice history 
- editing the AI process output files
- publishing the final output to the marketplace 

The backend does
- run the AI processing using an AI agentic system
  - load the audio recordings
  - transcribe them using a local STT model from huggingface
  - store the transcripts and mark their processing status
  - create a graph from the textual recordings
  - load the graph into a graph database
  - identify the main storylines in the text
  - use this to organize the transcripts into chapters taking into account the optional user annotation
  - annotate the text chunks with chapter numbers
  - for each chapter load all text chunk and use them to write the chapter
  - formulate relevant follow up questions based on the graph 
  - store the output text in markdown format 
- manage data for the UI display and marketplace
- 

### Key Features
- User authentication (login/registration)
- Audio recording with high-quality output
- Chapter-based organization system
- Topics and themes management
- Cloud storage integration
- User settings and profile management
- Payment integration for premium features

## Technology Stack

### Mobile (Flutter)
- **Framework**: Flutter 3.x
- **State Management**: Riverpod 2.x
- **Navigation**: go_router
- **Authentication**: Firebase Auth
- **Storage**: Firebase Storage
- **Audio Recording**: record package
- **HTTP Client**: Dio
- **Secure Storage**: flutter_secure_storage

### Web Frontend
- extended mobile also in flutter but for web

### Backend
- **Platform**: Python
- **Purpose**: Data storage, manipulation, and AI components
- **Integration**: RESTful API endpoints


### Cloud Services
- **Firebase Core**: App initialization and configuration
- **Firebase Auth**: User authentication and session management
- **Firebase Storage**: Audio file storage and retrieval

## Current Implementation Status of MOBILE

### ✅ Completed Components

#### 1. Project Structure
```
lib/
├── features/
│   ├── auth/
│   │   ├── auth_provider.dart          # Authentication state management
│   │   ├── login_screen.dart           # Login UI and logic
│   │   └── register_screen.dart        # Registration UI and logic
│   ├── audio/
│   │   ├── audio_recording_provider.dart # Audio recording state management
│   │   └── recording_screen.dart       # Recording UI and controls
│   └── chapter/
│       ├── chapter_provider.dart       # Chapter and audio clip data models
│       ├── chapter_list_screen.dart    # Main chapters listing
│       └── chapter_detail_screen.dart  # Individual chapter management
├── services/
│   ├── storage_service.dart           # Firebase Storage integration
│   └── api_service.dart              # Backend API communication
└── main.dart                         # App entry point and routing
```

#### 2. Authentication System
- **Provider**: `AuthNotifier` with Riverpod StateNotifier
- **States**: Initial, Authenticated, Unauthenticated, Error
- **Features**: Email/password login, registration, logout
- **Firebase Integration**: Complete with auth state listening
- **Navigation**: Automatic routing based on auth state

#### 3. Audio Recording
- **Provider**: `AudioRecordingNotifier` with states for recording lifecycle
- **States**: Initial, Recording, Paused, Recorded, Error
- **Controls**: Start, Stop, Pause, Resume recording
- **Permissions**: Built-in permission handling
- **File Management**: Path-based audio file handling

#### 4. Chapter Management
- **Data Models**: 
  - `AudioClip`: id, path, title, duration, createdAt
  - `Chapter`: id, title, audioClipIds, topics, createdAt, updatedAt
- **Provider**: `ChapterListNotifier` for CRUD operations
- **Features**: Add, edit, delete chapters; reorder audio clips; topic management
- **UI**: List view with dialogs for chapter editing

#### 5. Navigation & Routing
- **Router**: go_router with path-based navigation
- **Routes**: 
  - `/login` - Authentication entry point
  - `/register` - User registration
  - `/home` - Main chapter listing (redirects to chapter list)
  - `/record` - Audio recording interface
  - `/chapter/:id` - Individual chapter details
- **Protection**: Route guards based on authentication state

#### 6. Cloud Services Integration
- **Firebase Storage**: Audio file upload/download with user-specific paths
- **API Service**: HTTP client for backend communication
- **Metadata Handling**: Audio clip metadata storage and retrieval

### 🚧 Partially Implemented

#### Firebase Configuration
- Core Firebase initialization in main()
- Auth and Storage services configured
- **Missing**: Firebase project configuration files (google-services.json, GoogleService-Info.plist)

#### Backend Integration
- API service structure created with Dio
- Endpoint methods defined for audio metadata and chapter management
- **Missing**: Actual backend URL configuration and authentication tokens

### ❌ Not Yet Implemented

#### 1. Payment System
- Initial fee collection before app usage
- Payment provider integration (Stripe, PayPal, etc.)
- Payment state management
- Premium features unlocking

#### 2. User Settings
- Profile management screen
- User preferences (audio quality, storage options)
- Account settings (name, email, password change)
- App settings (notifications, themes)

#### 3. Audio Playback
- Audio player controls
- Waveform visualization
- Playback speed controls
- Chapter-wide playlist functionality

#### 4. Enhanced Security
- Audio file encryption at rest
- Request signing for API calls
- Biometric authentication option
- Secure token storage and refresh

#### 5. Error Handling & UX
- Comprehensive error boundaries
- Loading states throughout the app
- Offline support and sync
- Upload progress indicators

#### 6. Data Persistence
- Local database for offline access
- Audio file caching
- Chapter synchronization
- Conflict resolution

## Dependencies

### Current pubspec.yaml
```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  firebase_core: ^2.24.2
  firebase_auth: ^4.16.0
  flutter_riverpod: ^2.4.9
  riverpod_annotation: ^2.3.3
  record: ^5.0.4
  go_router: ^13.0.1
  dio: ^5.4.0
  flutter_secure_storage: ^9.0.0
  
dev_dependencies:
  flutter_test:
    sdk: flutter
  riverpod_generator: ^2.3.9
  build_runner: ^2.4.8
  flutter_lints: ^3.0.1
```

### Additional Dependencies Needed
```yaml
# For payment processing
stripe_payment: ^latest
pay: ^latest

# For audio playback
audioplayers: ^latest
audio_waveforms: ^latest

# For local storage
sqflite: ^latest
hive: ^latest

# For Firebase Storage
firebase_storage: ^latest

# For image handling (profile pictures)
image_picker: ^latest
cached_network_image: ^latest

# For biometric auth
local_auth: ^latest

# For file encryption
encrypt: ^latest
```

## Architecture Patterns

### State Management (Riverpod)
- **Providers**: Global state containers
- **StateNotifiers**: Complex state management with immutable updates
- **Consumer Widgets**: Reactive UI updates
- **Ref**: Dependency injection and provider reading

### Clean Architecture Principles
- **Separation of Concerns**: Features are modularized
- **Dependency Inversion**: Services injected through providers
- **Single Responsibility**: Each class has one clear purpose

### Error Handling Strategy
```dart
sealed class Result<T> {
  const Result();
  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(String error) = Failure<T>;
}
```

## Security Considerations

### Current Security Measures
- Firebase Auth for user authentication
- Secure storage for sensitive data
- HTTPS for all API communications

### Required Security Enhancements
- Audio file encryption before cloud upload
- Request signing with JWT tokens
- Input validation and sanitization
- Rate limiting for API calls
- Audit logging for user actions

## Testing Strategy

### Unit Tests (Not Implemented)
- Provider logic testing
- Model validation
- Service layer testing
- Utility function testing

### Integration Tests (Not Implemented)
- Authentication flow testing
- Audio recording/playback testing
- Chapter management workflow
- API integration testing

### Widget Tests (Not Implemented)
- Screen rendering validation
- User interaction testing
- Navigation flow testing
- Error state handling

## Development Environment Setup

### Prerequisites
1. Flutter SDK 3.x installed
2. Firebase project created with Auth and Storage enabled
3. Android Studio / VS Code with Flutter extensions
4. Physical device or emulator for audio testing

### Configuration Steps
1. **Firebase Setup**:
   - Create Firebase project
   - Enable Authentication (Email/Password)
   - Enable Storage with security rules
   - Download configuration files
   - Add to android/app and ios/Runner

2. **Backend Configuration**:
   - Update API_BASE_URL in api_service.dart
   - Configure authentication headers
   - Set up CORS for web testing

3. **Platform Permissions**:
   ```xml
   <!-- Android (android/app/src/main/AndroidManifest.xml) -->
   <uses-permission android:name="android.permission.RECORD_AUDIO" />
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
   <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
   ```

   ```xml
   <!-- iOS (ios/Runner/Info.plist) -->
   <key>NSMicrophoneUsageDescription</key>
   <string>This app needs microphone access to record audio</string>
   ```

## Next Development Steps

### Phase 1: Core Completion (1-2 weeks)
1. **Firebase Configuration**
   - Set up Firebase project
   - Add configuration files
   - Test authentication flow

2. **Audio Playback Implementation**
   - Add audioplayers dependency
   - Create audio player provider
   - Implement playback controls in chapter detail

3. **Error Handling Enhancement**
   - Add Result pattern
   - Implement error boundaries
   - Add loading states

### Phase 2: User Experience (1 week)
1. **Payment Integration**
   - Choose payment provider
   - Implement payment flow
   - Add payment state management

2. **Settings Screen**
   - User profile management
   - App preferences
   - Account settings

3. **Upload Progress & Offline Support**
   - Progress indicators
   - Local storage implementation
   - Sync mechanism

### Phase 3: Polish & Security (1 week)
1. **Security Enhancements**
   - File encryption
   - Biometric authentication
   - API security improvements

2. **Testing Implementation**
   - Unit tests for providers
   - Integration tests for flows
   - Widget tests for UI

3. **Performance Optimization**
   - Audio compression
   - Caching strategies
   - Memory management

## Common Issues & Solutions

### Audio Recording Issues
- **Permission Denied**: Ensure proper permissions in platform manifests
- **Recording Fails**: Check microphone availability and app permissions
- **File Path Issues**: Use proper path resolution for different platforms

### Firebase Integration Issues
- **Auth Not Working**: Verify configuration files are properly placed
- **Storage Upload Fails**: Check Firebase Storage rules and authentication
- **Build Errors**: Ensure all Firebase dependencies are compatible

### State Management Issues
- **Provider Not Updating**: Ensure StateNotifier properly notifies listeners
- **Memory Leaks**: Properly dispose controllers and listeners
- **Circular Dependencies**: Use proper provider hierarchy

## API Documentation

### Expected Backend Endpoints

#### Authentication
```
POST /auth/login
POST /auth/register
POST /auth/refresh
DELETE /auth/logout
```

#### User Management
```
GET /user/profile
PUT /user/profile
DELETE /user/account
```

#### Audio Management
```
POST /audio - Upload audio metadata
GET /audio/:id - Get audio details
PUT /audio/:id - Update audio metadata
DELETE /audio/:id - Delete audio file
```

#### Chapter Management
```
GET /chapters - List user chapters
POST /chapters - Create new chapter
GET /chapters/:id - Get chapter details
PUT /chapters/:id - Update chapter
DELETE /chapters/:id - Delete chapter
```

#### Payment
```
POST /payment/intent - Create payment intent
POST /payment/confirm - Confirm payment
GET /payment/history - Payment history
```

## Database Schema (Backend)

### Users Table
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    subscription_status VARCHAR(50) DEFAULT 'free'
);
```

### Chapters Table
```sql
CREATE TABLE chapters (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    topics TEXT[], -- Array of topic strings
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Audio Clips Table
```sql
CREATE TABLE audio_clips (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    chapter_id UUID REFERENCES chapters(id),
    title VARCHAR(255) NOT NULL,
    file_url TEXT NOT NULL,
    duration_seconds INTEGER,
    file_size_bytes BIGINT,
    order_index INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Deployment Considerations

### Flutter Build Configuration
- Configure app signing for release builds
- Set up build flavors for development/staging/production
- Configure ProGuard rules for Android
- Set up iOS distribution certificates

### Backend Deployment
- Set up CI/CD pipeline
- Configure environment variables
- Set up monitoring and logging
- Configure backup strategies

### Firebase Configuration
- Set up multiple environments (dev/staging/prod)
- Configure security rules
- Set up monitoring and alerts
- Configure backup policies

This document should be updated as the project evolves and new requirements are identified.
