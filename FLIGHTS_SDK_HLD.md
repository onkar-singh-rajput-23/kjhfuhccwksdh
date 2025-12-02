# High-Level Design (HLD) Diagram: Flights App & Flights SDK Architecture

## Overview

This document provides a comprehensive High-Level Design diagram showing the complete architecture of the Flights App and its integration with the Flights SDK.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    FLIGHTS APP (Main Application)                                           │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              APP ENTRY POINT & INITIALIZATION                                       │  │
│  │  ┌──────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐                       │  │
│  │  │   AppDelegate    │───▶│ ApplicationLaunch     │───▶│   WindowManager     │                       │  │
│  │  │  @UIApplication  │    │     Manager           │    │   (Window Setup)    │                       │  │
│  │  └──────────────────┘    └──────────────────────┘    └──────────────────────┘                       │  │
│  │         │                          │                            │                                   │  │
│  │         │                          ▼                            ▼                                   │  │
│  │         │              ┌──────────────────────────┐   ┌──────────────────────┐                      │  │
│  │         │              │  FlightsSDKIntegration   │   │   TabViewController  │                      │  │
│  │         │              │      .setup()            │   │   (Root Controller)  │                      │  │
│  │         │              └──────────────────────────┘   └──────────────────────┘                      │  │
│  │         │                          │                            │                                   │  │
│  │         │                          ▼                            ▼                                   │  │
│  │         │              ┌──────────────────────────┐   ┌──────────────────────┐                      │  │
│  │         │              │  Configure SDK with      │   │  ┌──────────────────┐ │                     │  │
│  │         │              │  Protocol Delegates:     │   │  │  Home Tab        │ │                     │  │
│  │         │              │  • Auth                  │   │  │  (Main App)      │ │                     │  │
│  │         │              │  • Remote Config         │   │  └──────────────────┘ │                     │  │
│  │         │              │  • Network Headers       │   │  ┌──────────────────┐ │                     │  │
│  │         │              │  • Analytics             │   │  │  Flights Tab     │ │                     │  │
│  │         │              │  • Deep Linking          │   │  │  (From SDK)     │  │                     │  │
│  │         │              │  • App Launcher          │   │  └──────────────────┘ │                     │  │
│  │         │              └──────────────────────────┘   └──────────────────────┘                      │  │
│  │         │                          │                            │                                   │  │
│  │         │                          └────────────┬───────────────┘                                   │  │
│  │         │                                         ▼                                                 │  │
│  │         │                          ┌──────────────────────────┐                                     │  │
│  │         │                          │   ixigoFlightsSDK        │                                     │  │
│  │         │                          │      .shared             │                                     │  │
│  │         │                          │   .configure(config)     │                                     │  │
│  │         │                          └──────────────────────────┘                                     │  │
│  │         │                                        │                                                  │  │
│  └─────────┼────────────────────────────────────────┼────────────────────────────────────────────────────┘  │
│            │                                        │                                                       │
│            │                                        │                                                       │
│  ┌─────────▼────────────────────────────────────────▼─────────────────────────────────────────────────────┐  │
│  │                              MAIN APP SERVICES & INFRASTRUCTURE                                        │  │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐            │  │
│  │  │  LoginManager    │  │  RemoteConfig    │  │ AnalyticsCenter  │  │ NetworkClientV2  │            │  │
│  │  │  (Auth State)    │  │  (Feature Flags) │  │  (CleverTap,     │  │  (Network Layer) │            │  │
│  │  └──────────────────┘  │                  │  │   Firebase,      │  └──────────────────┘            │  │
│  │         │              └──────────────────┘  │   Adjust)        │         │                        │  │
│  │         │                    │               └──────────────────┘         │                        │  │
│  │         │                    │                      │                      │                        │  │
│  │         └────────────────────┼──────────────────────┼──────────────────────┘                        │  │
│  │                              │                      │                                               │  │
│  │                              ▼                      ▼                                               │  │
│  │                    ┌──────────────────────────────────────┐                                         │  │
│  │                    │   Protocol Delegates (Bridges)      │                                         │  │
│  │                    │  • FlightsSDKAuthDelegate           │                                         │  │
│  │                    │  • FlightsSDKRemoteConfigDelegate   │                                         │  │
│  │                    │  • FlightsSDKAnalyticsProvider      │                                         │  │
│  │                    │  • FlightsSDKHeaderDelegate         │                                         │  │
│  │                    └──────────────────────────────────────┘                                         │  │
│  │                              │                                                                    │  │
│  └──────────────────────────────┼────────────────────────────────────────────────────────────────────┘  │
│                                 │                                                                        │
│  ┌──────────────────────────────▼─────────────────────────────────────────────────────────────────────┐  │
│  │                              MAIN APP VIEWS & VIEW MODELS                                           │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Home Tab (Main App)                                                                          │ │  │
│  │  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                         │ │  │
│  │  │  │ HomeRootView     │───▶│ HomeRootViewModel│───▶│  HomeClientV2    │                         │ │  │
│  │  │  │  (SwiftUI)       │    │  (@MainActor)    │    │  (API Calls)     │                         │ │  │
│  │  │  └──────────────────┘    └──────────────────┘    └──────────────────┘                         │ │  │
│  │  │         │                        │                        │                                    │ │  │
│  │  │         │                        ▼                        │                                    │ │  │
│  │  │         │              ┌──────────────────┐               │                                    │ │  │
│  │  │         │              │ HomeWidgetViewModel│             │                                    │ │  │
│  │  │         │              │   (Enum Wrapper)   │             │                                    │ │  │
│  │  │         │              └──────────────────┘               │                                    │ │  │
│  │  │         │                        │                        │                                    │ │  │
│  │  │         │                        ▼                        │                                    │ │  │
│  │  │         │              ┌─────────────────────────────────┐│                                    │ │  │
│  │  │         │              │ Uses SDK Widgets:               ││                                    │ │  │
│  │  │         │              │ • ImageCarouselSectionView      ││                                    │ │  │
│  │  │         │              │ • ExploreMasonrySectionView     ││                                    │ │  │
│  │  │         │              │ • TrackFlightSectionView        ││                                    │ │  │
│  │  │         │              │ • PartnerOffers                 ││                                    │ │  │
│  │  │         │              │ • ExclusivePartnerOffers        ││                                    │ │  │
│  │  │         │              └─────────────────────────────────┘│                                    │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │                                                                                                   │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Other App Pages (Main App Owned)                                                             │ │  │
│  │  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                         │ │  │
│  │  │  │ SRPView          │───▶│ SRPViewModel     │───▶│ SRPClient        │                         │ │  │
│  │  │  │ (Search Results) │    │                  │    │ (API Calls)      │                         │ │  │
│  │  │  └──────────────────┘    └──────────────────┘    └──────────────────┘                         │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                         │ │  │
│  │  │  │ FlightDetailView │───▶│ FlightDetailVM   │───▶│ FlightDetailClient│                        │ │  │
│  │  │  │                  │    │                  │    │                  │                         │ │  │
│  │  │  └──────────────────┘    └──────────────────┘    └──────────────────┘                         │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                         │ │  │
│  │  │  │ BookingFlow      │───▶│ BookingViewModel │───▶│ BookingClient    │                         │ │  │
│  │  │  │                  │    │                  │    │                  │                         │ │  │
│  │  │  └──────────────────┘    └──────────────────┘    └──────────────────┘                         │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                    │
                                                    │ Protocol-Based Integration
                                                    │ (Dependency Injection)
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    FLIGHTS SDK (ixigoFlightsSDK)                                            │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              SDK CORE (Singleton Entry Point)                                       │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  ixigoFlightsSDK.shared                                                                      │ │  │
│  │  │  • configure(FlightsIntegrationConfig)                                                       │ │  │
│  │  │  • getFlightsHomeViewController()                                                             │ │  │
│  │  │  • Stores protocol delegates from host app                                                    │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │         │                                                                                          │  │
│  │         ├──────────────────────────────────────────────────────────────────────────────────────┐  │  │
│  │         │                                                                                        │  │
│  │         ▼                          ▼                          ▼                                  │  │
│  │  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐                            │  │
│  │  │ FlightsAuth  │         │ FlightsRemote│         │ Flights      │                            │  │
│  │  │ Center       │         │ ConfigCenter │         │ Analytics    │                            │  │
│  │  │ (Auth State) │         │ (Config)     │         │ (Events)     │                            │  │
│  │  └──────────────┘         └──────────────┘         └──────────────┘                            │  │
│  │         │                          │                      │                                     │  │
│  │         │                          │                      │                                     │  │
│  │         └──────────────────────────┼──────────────────────┘                                     │  │
│  │                                    │                                                             │  │
│  │                                    ▼                                                             │  │
│  │                          ┌──────────────────┐                                                    │  │
│  │                          │ Protocol         │                                                    │  │
│  │                          │ Delegates        │                                                    │  │
│  │                          │ (From Host App)  │                                                    │  │
│  │                          └──────────────────┘                                                    │  │
│  └────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              SDK FEATURES - HOME PAGE                                                │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Flights Tab (SDK Owned)                                                                     │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │ FlightsHomeHostingViewController (UIKit)                                              │ │ │  │
│  │  │  │         │                                                                             │ │ │  │
│  │  │  │         ▼                                                                             │ │ │  │
│  │  │  │  ┌─────────────────────────────────────────────────────────────────────────────────┐ │ │ │  │
│  │  │  │  │ FlightsHomeView (SwiftUI)                                                      │ │ │ │  │
│  │  │  │  │         │                                                                      │ │ │ │  │
│  │  │  │  │         ▼                                                                      │ │ │ │  │
│  │  │  │  │  ┌──────────────────────────────────────────────────────────────────────────┐ │ │ │ │  │
│  │  │  │  │  │ FlightsHomeViewModel (@MainActor, ObservableObject)                      │ │ │ │ │  │
│  │  │  │  │  │  • @Published var whyBookItems: [WhyBookItem]                            │ │ │ │ │  │
│  │  │  │  │  │  • @Published var widgets: [FlightsHomeWidgetViewModel]                  │ │ │ │ │  │
│  │  │  │  │  │  • @Published var isLoading: Bool                                       │ │ │ │ │  │
│  │  │  │  │  │         │                                                                │ │ │ │ │  │
│  │  │  │  │  │         ├──────────────────────────────────────────────────────────────┐ │ │ │ │ │  │
│  │  │  │  │  │         │                                                              │ │ │ │ │ │  │
│  │  │  │  │  │         ▼                          ▼                                   │ │ │ │ │ │  │
│  │  │  │  │  │  ┌──────────────┐         ┌──────────────────────┐                  │ │ │ │ │ │  │
│  │  │  │  │  │  │ FlightHome    │         │ FlightsHomeWidgets   │                  │ │ │ │ │ │  │
│  │  │  │  │  │  │ Client        │         │ Service              │                  │ │ │ │ │ │  │
│  │  │  │  │  │  │ (WhyBook API) │         │ (Widgets API)        │                  │ │ │ │ │ │  │
│  │  │  │  │  │  └──────────────┘         └──────────────────────┘                  │ │ │ │ │ │  │
│  │  │  │  │  │         │                          │                                  │ │ │ │ │ │  │
│  │  │  │  │  │         └──────────┬───────────────┘                                  │ │ │ │ │ │  │
│  │  │  │  │  │                    ▼                                                   │ │ │ │ │ │  │
│  │  │  │  │  │          ┌──────────────────┐                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │ NetworkManagerV2 │                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │  (Shared Instance)│                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │  • Uses host app  │                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │    headers        │                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │  • Uses host app  │                                         │ │ │ │ │ │  │
│  │  │  │  │  │          │    auth           │                                         │ │ │ │ │ │  │
│  │  │  │  │  │          └──────────────────┘                                         │ │ │ │ │ │  │
│  │  │  │  │  └──────────────────────────────────────────────────────────────────────────┘ │ │ │ │  │
│  │  │  │  └─────────────────────────────────────────────────────────────────────────────────┘ │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │                                                                                                     │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  SDK WIDGETS (Reusable Components)                                                            │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget ViewModels (Business Logic)                                                     │ │ │  │
│  │  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │ │ │  │
│  │  │  │  │ Partner      │  │ Exclusive    │  │ Explore      │  │ TrackFlight  │            │ │ │  │
│  │  │  │  │ OffersVM     │  │ Partner      │  │ MasonryVM    │  │ SectionVM    │            │ │ │  │
│  │  │  │  └──────────────┘  │ OffersVM     │  └──────────────┘  └──────────────┘            │ │ │  │
│  │  │  │         │          └──────────────┘         │                  │                    │ │ │  │
│  │  │  │         │                │                   │                  │                    │ │ │  │
│  │  │  │         ▼                ▼                   ▼                  ▼                    │ │ │  │
│  │  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │ │  │
│  │  │  │  │ Partner      │  │ Exclusive    │  │ Explore       │  │ TrackFlight   │         │ │ │  │
│  │  │  │  │ OffersView   │  │ Partner      │  │ Masonry       │  │ SectionView  │         │ │ │  │
│  │  │  │  │              │  │ OffersView   │  │ SectionView   │  │              │         │ │ │  │
│  │  │  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘         │ │ │  │
│  │  │  │                                                                                    │ │ │  │
│  │  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │ │  │
│  │  │  │  │ ImageCarousel│  │ FlagShip     │  │ WhatsNew     │  │ RecentSearches│         │ │ │  │
│  │  │  │  │ SectionView  │  │ View         │  │ View         │  │ SectionView  │         │ │ │  │
│  │  │  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘         │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │                                                                                                     │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  SDK SERVICES & UTILITIES                                                                    │ │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │ │  │
│  │  │  │ Explore      │  │ FlightsDeep  │  │ AssetCache   │  │ Flights      │                  │ │  │
│  │  │  │ Masonry      │  │ LinkRouter   │  │ (Image Cache)│  │ Analytics    │                  │ │  │
│  │  │  │ Service      │  │ (Navigation) │  │              │  │ (Event      │                  │ │  │
│  │  │  │              │  │              │  │              │  │  Tracking)   │                  │ │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘                  │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │                                                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                    │
                                                    │ Shared Network Layer
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              SHARED INFRASTRUCTURE (ixigoNetworkSDK)                                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  NetworkManagerV2 (Singleton)                                                                        │  │
│  │  • Configured by Main App (SecurityData, Headers)                                                   │  │
│  │  • Used by both Main App and SDK                                                                    │  │
│  │  • Handles all HTTP requests                                                                        │  │
│  │  • Uses protocol delegates for headers/auth from host app                                           │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow & Coordination

### 1. Initialization Flow
```
AppDelegate 
  → ApplicationLaunchManager 
  → FlightsSDKIntegration.setup() 
  → ixigoFlightsSDK.shared.configure(config) 
  → SDK stores protocol delegates
```

### 2. View Display Flow
```
TabViewController 
  → Flights Tab 
  → ixigoFlightsSDK.shared.getFlightsHomeViewController()
  → FlightsHomeHostingViewController 
  → FlightsHomeView 
  → FlightsHomeViewModel
```

### 3. Data Fetching Flow (SDK)
```
FlightsHomeViewModel.init() 
  → FlightHomeClient.fetchWhyBook() 
  → NetworkManagerV2.request() 
  → API Server 
  → Response 
  → ViewModel updates @Published properties 
  → View re-renders
```

### 4. Widget Usage Flow (Main App using SDK Widgets)
```
HomeRootView 
  → HomeWidgetViewModel (enum) 
  → Switch on widget type 
  → SDK Widget View (e.g., ImageCarouselSectionView) 
  → SDK Widget ViewModel 
  → SDK Service 
  → NetworkManagerV2 
  → API
```

### 5. Coordination Mechanisms
- **@Published properties**: SwiftUI reactive updates
- **@StateObject**: View owns ViewModel
- **@ObservedObject**: View observes ViewModel
- **Protocol delegates**: SDK ↔ Main App communication
- **NotificationCenter**: Auth state changes, Remote Config updates
- **Deep Link Router**: Navigation coordination

---

## Key Architectural Patterns

1. **Dependency Injection**: Protocol-based delegates allow SDK to use host app services
2. **Singleton Pattern**: `ixigoFlightsSDK.shared`, `NetworkManagerV2.shared`
3. **MVVM Architecture**: Clear separation of Views, ViewModels, and Models
4. **Reactive Programming**: SwiftUI @Published properties for state management
5. **Modular Design**: SDK is self-contained but integrates via protocols
6. **Shared Infrastructure**: Network layer shared between app and SDK

---

## Component Details

### Main App Components

#### Entry Point
- **AppDelegate**: iOS app entry point with `@UIApplicationMain`
- **ApplicationLaunchManager**: Orchestrates app initialization
- **WindowManager**: Manages app window and root view controller

#### Services
- **LoginManager**: Manages user authentication state
- **RemoteConfig**: Handles feature flags and remote configuration
- **AnalyticsCenter**: Central analytics hub (CleverTap, Firebase, Adjust)
- **NetworkClientV2**: Network layer facade

#### Views & ViewModels
- **HomeRootView/ViewModel**: Main app home screen
- **SRPView/ViewModel**: Search Results Page
- **FlightDetailView/ViewModel**: Flight detail page
- **BookingFlow/ViewModel**: Booking process

### SDK Components

#### Core
- **ixigoFlightsSDK**: Singleton entry point
- **FlightsIntegrationConfig**: Configuration container
- **Protocol Delegates**: Integration interfaces

#### Features
- **FlightsHomeView/ViewModel**: SDK home screen
- **FlightsHomeWidgetsService**: Widget data service
- **FlightHomeClient**: API client for home data

#### Widgets
- **ImageCarouselSectionView**: Image carousel widget
- **ExploreMasonrySectionView**: Explore destinations widget
- **TrackFlightSectionView**: Flight tracking widget
- **PartnerOffers/ExclusivePartnerOffers**: Offer widgets
- **FlagShip, WhatsNew, RecentSearches**: Additional widgets

#### Utilities
- **FlightsDeepLinkRouter**: Deep linking navigation
- **AssetCache**: Image caching utility
- **FlightsAnalytics**: Analytics wrapper

### Shared Infrastructure

- **NetworkManagerV2**: Shared network layer used by both app and SDK
- **ixigoNetworkSDK**: Network SDK package
- **ixigoUI**: Shared UI components
- **ixigoDesignSDK**: Design system SDK

---

## Integration Points

1. **SDK Configuration**: `FlightsSDKIntegration.setup()` configures SDK with protocol delegates
2. **View Integration**: `TabViewController` uses `ixigoFlightsSDK.shared.getFlightsHomeViewController()`
3. **Widget Integration**: `HomeRootView` uses SDK widgets via `HomeWidgetViewModel` enum
4. **Network Integration**: Both app and SDK use shared `NetworkManagerV2`
5. **Analytics Integration**: SDK events routed through main app's `AnalyticsCenter`
6. **Auth Integration**: SDK checks auth state via `FlightsSDKAuthDelegate`
7. **Deep Link Integration**: SDK deep links handled by `FlightsDeepLinkRouter`

---

## File Locations

### Main App
- Entry: `Flights/Flights/Common/Application/AppDelegate.swift`
- Integration: `Flights/Flights/Epics/Flights/Integration/FlightsSDKIntegration.swift`
- Home: `Flights/Flights/NewCo/Home/`
- SRP: `Flights/Flights/NewCo/SRP/`

### SDK
- Core: `Packages/ixigoFlightsSDK/Sources/ixigoFlightsSDK/ixigoFlightsSDK.swift`
- Protocols: `Packages/ixigoFlightsSDK/Sources/ixigoFlightsSDK/Protocols/`
- Features: `Packages/ixigoFlightsSDK/Sources/ixigoFlightsSDK/Features/`
- Widgets: `Packages/ixigoFlightsSDK/Sources/ixigoFlightsSDK/Features/Home/Widgets/`

