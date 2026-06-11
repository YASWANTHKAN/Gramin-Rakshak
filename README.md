# Gramin Rakshak - Rural Health Guard

A mobile health assistant designed for rural India that provides offline-capable fever triage, first aid guidance, and emergency services access using voice-first interaction in local languages.

## Problem Statement

Rural communities in India face limited access to healthcare facilities, language barriers, and poor internet connectivity. When illness strikes, villagers often lack basic guidance on symptom severity, when to seek emergency care, and where to find the nearest health center.

## Solution

Gramin Rakshak (Village Protector) is a Flutter-based mobile app that acts as a rural health companion. It uses a **Bayesian triage engine** to assess symptoms through a conversational interface (voice or text), provides first aid steps, locates the nearest Primary Health Centre (PHC), and works entirely offline.

## Features

### Conversational Symptom Triage
- Voice-based symptom input using Speech-to-Text (supports English, Hindi, Telugu, Kannada, Malayalam)
- Text-to-Speech output for accessibility
- Bayesian likelihood-ratio engine that scores symptoms against 10 endemic conditions
- Red-flag detection for life-threatening symptoms with immediate escalation

### Diseases Covered
| Disease | ICD Code |
|---------|----------|
| Malaria | B50-B54 |
| Dengue | A90-A91 |
| Typhoid | A01.0 |
| Chikungunya | A92.0 |
| Viral Fever | A94 |
| Leptospirosis | A27 |
| Scrub Typhus | A75.3 |
| Japanese Encephalitis | A83.0 |
| Influenza | J10-J11 |
| Enteric Fever | A01.1 |

### PHC Locator
- GPS-based nearest health centre finder
- Interactive map with OpenStreetMap tiles
- Search and filter by state/district
- One-tap navigation and calling

### Emergency Services
- One-tap dial for 108 (Ambulance), 112 (Police), 104 (Health Helpline), 1098 (Child Helpline), 181 (Women Helpline)
- Shows nearest PHC phone based on user's registered district
- Works offline (phone dialer)

### First Aid Guide
- Step-by-step instructions for rural emergencies (snake bite, burns, drowning, fractures, etc.)
- Clear "Do" and "Do NOT" lists
- Accessible without internet

### Body Map
- Tap body regions to see common problems, home remedies, and warning signs
- Severity classification (low / moderate / high)

### Maternal Health Tracker
- Trimester-wise guidance (what's happening, warning signs, PHC visit schedule, diet tips)
- Milestone tracking for pregnancies

### Daily Health Log
- Quick daily check-in: sleep, water, pain, mood, food, medicine
- Visual health score and 7-day trends
- Streak tracking for engagement

### Medicine Reminders
- Add medicines with time-of-day slots (morning, afternoon, evening, night)
- Local push notifications
- Track adherence

### Seasonal Health Tips
- Season-aware disease prevention advice (summer, monsoon, post-monsoon, winter)
- Home remedies and warning signs per season

### Multi-Language Support
- English, Hindi, Telugu, Kannada, Malayalam
- Entire UI and triage output translated
- Voice I/O in all supported languages

### Offline-First Architecture
- Hive local database for triage history
- Knowledge graph embedded in-app (no API calls needed for triage)
- Firebase Firestore sync when online (optional)
- Graceful offline fallback on Firebase failure

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Flutter (Dart) |
| State Management | Provider |
| Local Storage | Hive, SharedPreferences |
| Cloud Backend | Firebase Auth, Cloud Firestore |
| Voice I/O | speech_to_text, flutter_tts |
| Maps | flutter_map + OpenStreetMap |
| Location | Geolocator |
| Notifications | flutter_local_notifications |
| UI | Material 3, Google Fonts, flutter_animate |

## Project Structure

```
lib/
├── main.dart                  # App entry point, Firebase init, Provider setup
├── firebase_options.dart      # Firebase configuration
├── models/
│   └── models.dart            # Data models (TriageRecord, Symptom, Condition, etc.)
├── data/
│   └── knowledge_graph.dart   # Disease knowledge base (symptoms, conditions, PHC list)
├── engine/
│   ├── triage_engine.dart     # Bayesian + weighted triage scoring engine
│   ├── context_engine.dart    # Contextual follow-up question logic
│   ├── speech_service.dart    # Speech-to-Text wrapper
│   ├── tts_service.dart       # Text-to-Speech wrapper
│   ├── language_service.dart  # i18n translations (5 languages)
│   ├── language_provider.dart # Language state management
│   ├── theme_provider.dart    # Dark/light theme state
│   ├── firestore_service.dart # Cloud Firestore CRUD
│   ├── analytics_service.dart # Usage analytics
│   └── notification_service.dart # Local notification scheduling
├── screens/
│   ├── onboarding_screen.dart      # First-time user setup
│   ├── login_screen.dart           # Phone-based login
│   ├── signup_screen.dart          # User registration
│   ├── main_shell.dart             # Bottom navigation shell
│   ├── conversation_screen.dart    # Voice/text symptom triage chat
│   ├── triage_result_screen.dart   # Diagnosis results + first aid
│   ├── body_map_screen.dart        # Interactive body part selector
│   ├── phc_locator_screen.dart     # Map-based PHC finder
│   ├── emergency_screen.dart       # Emergency numbers + quick dial
│   ├── first_aid_screen.dart       # First aid instructions
│   ├── maternal_health_screen.dart # Pregnancy tracker
│   ├── daily_health_screen.dart    # Daily health check-in
│   ├── medicine_reminder_screen.dart # Medicine scheduling
│   ├── seasonal_tips_screen.dart   # Season-wise health tips
│   ├── history_screen.dart         # Past triage records
│   └── profile_screen.dart         # User profile & settings
└── theme/
    └── app_theme.dart         # Material 3 theme configuration
```

## Getting Started

### Prerequisites
- Flutter SDK >= 3.0.0
- Dart SDK >= 3.0.0
- Android Studio / VS Code with Flutter extension
- (Optional) Firebase project for cloud sync

### Installation

```bash
# Clone the repository
git clone https://github.com/YASWANTHKAN/Gramin-Rakshak.git
cd Gramin-Rakshak

# Install dependencies
flutter pub get

# Generate Hive adapters
flutter pub run build_runner build

# Run on device/emulator
flutter run
```

### Firebase Setup (Optional)
The app works fully offline without Firebase. To enable cloud sync:
1. Create a Firebase project at [console.firebase.google.com](https://console.firebase.google.com)
2. Add Android/iOS apps and download config files
3. Replace `lib/firebase_options.dart` with your generated config

## How the Triage Engine Works

1. **Symptom Collection** - User describes symptoms via voice (in any supported language) or guided questions
2. **NLP Extraction** - Multi-language keyword matching extracts symptom IDs from free text
3. **Context Questions** - Follow-up questions for red-flag symptoms (e.g., "Is this the worst headache you've ever had?")
4. **Dual Scoring** -
   - *Weighted scoring*: each symptom contributes a weight to each condition, with red-flag bonuses
   - *Bayesian scoring*: log-likelihood ratios compute posterior probability per condition
5. **Consensus** - If Bayesian and weighted engines disagree significantly (log-odds diff > 2.0), the Bayesian result wins
6. **Severity Classification** - Low / Moderate / High based on score percentage and red flags
7. **Output** - Condition name, ICD code, matched symptoms, first aid steps, warning signs, and next steps

## Supported Languages

| Code | Language | Voice I/O |
|------|----------|-----------|
| en | English | Yes |
| hi | Hindi | Yes |
| te | Telugu | Yes |
| kn | Kannada | Yes |
| ml | Malayalam | Yes |

## Screenshots

*Coming soon*

## Disclaimer

Gramin Rakshak is a health awareness and triage assistance tool. It is **NOT** a substitute for professional medical diagnosis or treatment. Always consult a qualified healthcare provider for medical decisions. In emergencies, call 108 immediately.

## License

This project is open source. See the repository for license details.

## Author

**Yaswanth Kan** - [GitHub](https://github.com/YASWANTHKAN)
