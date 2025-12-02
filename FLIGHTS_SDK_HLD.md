# High-Level Design (HLD) Diagram: Flights App & Flights SDK Architecture

## Overview

This document provides a comprehensive High-Level Design diagram showing the complete architecture of the Flights App and its integration with the Flights SDK.

---
# High-Level Design (HLD): Flights SDK (ixigoFlightsSDK) - Internal Architecture

## Overview

This document provides a comprehensive High-Level Design diagram focused exclusively on the **Flights SDK (ixigoFlightsSDK)** internal architecture, structure, components, and data flow patterns.

---

## SDK Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    FLIGHTS SDK (ixigoFlightsSDK)                                            │
│                                    Swift Package Manager (SPM) Package                                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 1: SDK CORE (Entry Point)                                         │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  ixigoFlightsSDK (Singleton)                                                                 │ │  │
│  │  │  • shared: Static singleton instance                                                         │ │  │
│  │  │  • config: FlightsIntegrationConfig (stores protocol delegates)                              │ │  │
│  │  │  • configure(_ config: FlightsIntegrationConfig)                                             │ │  │
│  │  │    - Stores protocol delegates from host app                                                 │ │  │
│  │  │    - Configures FlightsAnalytics.analyticsProvider                                           │ │  │
│  │  │    - Configures NetworkExtraHeaders.provider                                                 │ │  │
│  │  │    - Configures FlightsDeepLinkRouter.shared.interceptor                                     │ │  │
│  │  │    - Configures IxigoUI theme                                                                │ │  │
│  │  │  • getFlightsHomeViewController() → FlightsHomeHostingViewController                        │ │  │
│  │  │  • remoteConfigValue(forKey:) → Gets value from host app delegate                           │ │  │
│  │  │  • openURL(_:presenter:onQuit:) → Delegates to host app launcher                             │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  │         │                                                                                          │  │
│  │         ▼                                                                                          │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  FlightsIntegrationConfig (Configuration Container)                                           │ │  │
│  │  │  • authDelegate: FlightsAuthDelegate?                                                        │ │  │
│  │  │  • remoteConfigDelegate: FlightsRemoteConfigDelegate?                                         │ │  │
│  │  │  • networkHeaderDelegate: FlightsNetworkHeaderDelegate?                                       │ │  │
│  │  │  • networkSecureDataDelegate: FlightsSecuredDataDelegate?                                    │ │  │
│  │  │  • networkInterceptorDelegate: FlightsNetworkInterceptorDelegate?                            │ │  │
│  │  │  • appLauncherDelegate: FlightsAppLauncherDelegate?                                          │ │  │
│  │  │  • analyticsDelegate: FlightsAnalyticsProviderDelegate?                                       │ │  │
│  │  │  • extraHeadersProvider: ExtraHeadersProvider?                                               │ │  │
│  │  │  • deeplinkInterceptor: FlightsExternalDeepLinkInterceptor?                                   │ │  │
│  │  │  • theme: ThemePreset (.brand, .trains, .abhibus, .confirmtkt)                               │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 2: PROTOCOL LAYER (Integration Interfaces)                    │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  FlightsSDKIntegrationProtocols.swift                                                          │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐              │ │  │
│  │  │  │ FlightsAuthDelegate  │  │ FlightsRemoteConfig  │  │ FlightsNetworkHeader │              │ │  │
│  │  │  │ • flightsSDKIsUser  │  │ Delegate             │  │ Delegate             │              │ │  │
│  │  │  │   LoggedIn() → Bool  │  │ • flightsSDKRemote  │  │ • flightsSDKAdditional│             │ │  │
│  │  │  └──────────────────────┘  │   ConfigValue(...)   │  │   Headers(...)       │              │ │  │
│  │  │                             └──────────────────────┘  └──────────────────────┘              │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐              │ │  │
│  │  │  │ FlightsSecuredData    │  │ FlightsNetwork        │  │ FlightsAppLauncher  │              │ │  │
│  │  │  │ Delegate              │  │ InterceptorDelegate   │  │ Delegate            │              │ │  │
│  │  │  │ • flightsSDKSecure    │  │ • flightsSDKWillSend │  │ • flightsSDKOpen(...)│             │ │  │
│  │  │  │   Headers(...)        │  │ • flightsSDKDidReceive│  └──────────────────────┘              │ │  │
│  │  │  └──────────────────────┘  └──────────────────────┘                                        │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐              │ │  │
│  │  │  │ FlightsAnalytics      │  │ ExtraHeadersProvider  │  │ FlightsExternalDeep  │              │ │  │
│  │  │  │ ProviderDelegate     │  │ • extraHeaders(forPath)│  │ LinkInterceptor      │              │ │  │
│  │  │  │ • recordEvent(...)   │  └──────────────────────┘  │ • handleExternal(...) │              │ │  │
│  │  │  │ • logError(...)      │                             └──────────────────────┘              │ │  │
│  │  │  │ • logWarning(...)    │                                                                    │ │  │
│  │  │  │ • logInfo(...)       │                                                                    │ │  │
│  │  │  │ • logDebug(...)      │                                                                    │ │  │
│  │  │  └──────────────────────┘                                                                    │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────────────────────────────────────────────────────────────┐           │ │  │
│  │  │  │  FlightsAuthCenter (Enum - Auth State Management)                            │           │ │  │
│  │  │  │  • isLoggedIn: Bool (computed from authDelegate)                             │           │ │  │
│  │  │  │  • notifyAuthStateChanged(_ isLoggedIn: Bool)                                 │           │ │  │
│  │  │  │    - Posts NotificationCenter notification                                    │           │ │  │
│  │  │  └──────────────────────────────────────────────────────────────────────────────┘           │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────────────────────────────────────────────────────────────┐           │ │  │
│  │  │  │  FlightsRemoteConfigCenter (Enum - Config Management)                         │           │ │  │
│  │  │  │  • notifyChanges(_ keys: [String])                                           │           │ │  │
│  │  │  │    - Posts NotificationCenter notification                                     │           │ │  │
│  │  │  └──────────────────────────────────────────────────────────────────────────────┘           │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌──────────────────────────────────────────────────────────────────────────────┐           │ │  │
│  │  │  │  NetworkExtraHeaders (Enum - Extra Headers Injection)                        │           │ │  │
│  │  │  │  • provider: ExtraHeadersProvider? (static)                                  │           │ │  │
│  │  │  └──────────────────────────────────────────────────────────────────────────────┘           │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 3: FEATURE LAYER (Home Feature)                                  │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Features/Home/                                                                              │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsHomeHostingViewController (UIKit)                                                │ │ │  │
│  │  │  │  • UIHostingController<FlightsHomeView>                                                 │ │ │  │
│  │  │  │  • Wraps FlightsHomeView for UIKit integration                                          │ │ │  │
│  │  │  │  • Entry point: ixigoFlightsSDK.shared.getFlightsHomeViewController()                   │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │         │                                                                                     │ │  │
│  │  │         ▼                                                                                     │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsHomeView (SwiftUI View)                                                          │ │ │  │
│  │  │  │  • Main SDK home screen view                                                            │ │ │  │
│  │  │  │  • Displays FlightsHomeSearchForm                                                        │ │ │  │
│  │  │  │  • Displays WhyBook section                                                              │ │ │  │
│  │  │  │  • Displays widgets from FlightsHomeViewModel                                           │ │ │  │
│  │  │  │  • Displays ExploreMasonrySectionView                                                    │ │ │  │
│  │  │  │  • Uses @StateObject for FlightsHomeViewModel                                           │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │         │                                                                                     │ │  │
│  │  │         ▼                                                                                     │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsHomeViewModel (@MainActor, ObservableObject)                                  │ │ │  │
│  │  │  │  • @Published var whyBookData: [WhybookV2]?                                            │ │ │  │
│  │  │  │  • @Published var isLoading: Bool                                                     │ │ │  │
│  │  │  │  • @Published var error: Error?                                                       │ │ │  │
│  │  │  │  • @Published var widgets: [FlightsHomeWidgetViewModel]                                │ │ │  │
│  │  │  │                                                                                          │ │ │  │
│  │  │  │  • init() → Calls loadWhyBookData() & loadWidgets()                                    │ │ │  │
│  │  │  │  • loadWhyBookData() → FlightHomeClient.fetchWhyBook()                                │ │ │  │
│  │  │  │  • loadWidgets() → FlightsHomeWidgetsService.fetchWidgets()                           │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │         │                                                                                     │ │  │
│  │  │         ├──────────────────────────────────────┬──────────────────────────────────────────┐ │ │  │
│  │  │         │                                      │                                          │ │ │  │
│  │  │         ▼                                      ▼                                          │ │ │  │
│  │  │  ┌──────────────────────┐         ┌──────────────────────────────────────────────┐     │ │ │  │
│  │  │  │  FlightHomeClient    │         │  FlightsHomeWidgetsService                    │     │ │ │  │
│  │  │  │  (API Client)        │         │  (Widget Service)                              │     │ │ │  │
│  │  │  │                      │         │                                                  │     │ │ │  │
│  │  │  │  • fetchWhyBook()    │         │  • fetchWidgets() async                         │     │ │ │  │
│  │  │  │    - GET /growth/api │         │    - Calls FlightHomeClient.fetchHomeWidgets() │     │ │ │  │
│  │  │  │      /v1/features    │         │    - Maps DTOs to ViewModels                    │     │ │ │  │
│  │  │  │    - Query: product= │         │    - Returns [FlightsHomeWidgetViewModel]      │     │ │ │  │
│  │  │  │      FLIGHTS,        │         │                                                  │     │ │ │  │
│  │  │  │      type=WHY_BOOK    │         │                                                  │     │ │ │  │
│  │  │  │                      │         │                                                  │     │ │ │  │
│  │  │  │  • fetchHomeWidgets() │         │                                                  │     │ │ │  │
│  │  │  │    - GET /widgets    │         │                                                  │     │ │ │  │
│  │  │  │    - Returns          │         │                                                  │     │ │ │  │
│  │  │  │      [FlightsHome    │         │                                                  │     │ │ │  │
│  │  │  │       WidgetDTO]     │         │                                                  │     │ │ │  │
│  │  │  └──────────────────────┘         └──────────────────────────────────────────────┘     │ │ │  │
│  │  │         │                                      │                                          │ │ │  │
│  │  │         └──────────────┬───────────────────────┘                                          │ │ │  │
│  │  │                        ▼                                                                   │ │ │  │
│  │  │              ┌──────────────────┐                                                          │ │ │  │
│  │  │              │ NetworkManagerV2 │                                                          │ │ │  │
│  │  │              │  (from ixigo    │                                                          │ │ │  │
│  │  │              │   NetworkSDK)   │                                                          │ │ │  │
│  │  │              │  • Shared       │                                                          │ │ │  │
│  │  │              │    instance     │                                                          │ │ │  │
│  │  │              │  • Uses protocol│                                                          │ │ │  │
│  │  │              │    delegates    │                                                          │ │ │  │
│  │  │              │    for headers   │                                                          │ │ │  │
│  │  │              └──────────────────┘                                                          │ │ │  │
│  │  │                                                                                               │ │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsHomeSearchForm (SwiftUI View)                                                 │ │ │  │
│  │  │  │  • Flight search form component                                                       │ │ │  │
│  │  │  │  • Handles user input for flight search                                                │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 4: WIDGET LAYER (Reusable Components)                            │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Features/Home/Widgets/                                                                      │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 1: PartnerOffers                                                         │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ PartnerOffers│───▶│ PartnerOffers│───▶│ PartnerOffers│                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │ View         │                              │ │ │  │
│  │  │  │  │ (Decodable)  │    │ (Business)   │    │ (SwiftUI)    │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 2: ExclusivePartnerOffers                                                  │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ Exclusive    │───▶│ Exclusive    │───▶│ Exclusive    │                              │ │ │  │
│  │  │  │  │ PartnerOffers│    │ PartnerOffers│    │ PartnerOffers│                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │ View         │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 3: ExploreMasonrySection                                                  │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │ │ │  │
│  │  │  │  │ Explore      │───▶│ Explore      │───▶│ Explore      │───▶│ Explore      │        │ │ │  │
│  │  │  │  │ Masonry      │    │ Masonry      │    │ Masonry      │    │ Masonry      │        │ │ │  │
│  │  │  │  │ Models       │    │ Service      │    │ ViewModel    │    │ SectionView  │        │ │ │  │
│  │  │  │  │              │    │ (API Client) │    │              │    │              │        │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘        │ │ │  │
│  │  │  │         │                    │                  │                  │                   │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┼──────────────────┘                   │ │ │  │
│  │  │  │                             ▼                                                          │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ MasonryGrid  │                                                     │ │ │  │
│  │  │  │                    │ (Layout)     │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 4: TrackFlightSection                                                     │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ TrackFlight  │───▶│ TrackFlight  │───▶│ TrackFlight  │                              │ │ │  │
│  │  │  │  │ CardModel    │    │ Section      │    │ SectionView  │                              │ │ │  │
│  │  │  │  │              │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                             │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ TrackFlight  │                                                     │ │ │  │
│  │  │  │                    │ CardView     │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 5: ImageCarouselSection                                                   │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ ImageCarousel│───▶│ ImageCarousel│───▶│ ImageCarousel│                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │ SectionView  │                              │ │ │  │
│  │  │  │  │              │    │              │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 6: FlagShip                                                                │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ FlagShip     │───▶│ Flagship     │───▶│ FlagShipView  │                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐    ┌──────────────┐                               │ │ │  │
│  │  │  │                    │ FlagShip     │    │ FlagShip     │                               │ │ │  │
│  │  │  │                    │ HeaderView   │    │ ItemView     │                               │ │ │  │
│  │  │  │                    └──────────────┘    └──────────────┘                               │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 7: WhatsNew                                                                │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ WhatsNew     │───▶│ WhatsNew     │───▶│ WhatsNewView  │                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │ │ │  │
│  │  │  │                    │ WhatsNew     │    │ WhatsNew     │    │ WhatsNew     │          │ │ │  │
│  │  │  │                    │ ItemView    │    │ ItemCell     │    │ Collection   │          │ │ │  │
│  │  │  │                    │              │    │              │    │ View         │          │ │ │  │
│  │  │  │                    └──────────────┘    └──────────────┘    └──────────────┘          │ │ │  │
│  │  │  │                                                                                          │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ WhatsNew     │                                                     │ │ │  │
│  │  │  │                    │ ItemViewModel│                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 8: RecentSearches                                                         │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ RecentSearch │───▶│ RecentSearch │───▶│ RecentSearch │                              │ │ │  │
│  │  │  │  │ Response     │    │ Response     │    │ esSectionView│                              │ │ │  │
│  │  │  │  │              │    │ Mapper       │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ RecentSearch │                                                     │ │ │  │
│  │  │  │                    │ esViewModel  │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 9: SpotlightSection                                                      │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ Spotlight    │───▶│ Spotlight    │───▶│ Spotlight     │                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │ SectionView   │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │ │ │  │
│  │  │  │                    │ Spotlight     │    │ Spotlight     │    │ Spotlight     │        │ │ │  │
│  │  │  │                    │ CardView       │    │ Filter       │    │ Horizontal   │        │ │ │  │
│  │  │  │                    │               │    │ Section      │    │ PanGesture   │        │ │ │  │
│  │  │  │                    └──────────────┘    └──────────────┘    └──────────────┘        │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 10: DealsOffer                                                             │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ DealsOffer   │───▶│ DealsOffer   │───▶│ DealsOfferView│                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ DealsOffer    │                                                     │ │ │  │
│  │  │  │                    │ ItemView     │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 11: ExperienceSection                                                     │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ Experience   │───▶│ Experience   │───▶│ Experience   │                              │ │ │  │
│  │  │  │  │ WidgetModel  │    │ Widget       │    │ Widget       │                              │ │ │  │
│  │  │  │  │              │    │ ViewModel    │    │ CardsView    │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ Experience   │                                                     │ │ │  │
│  │  │  │                    │ Widget       │                                                     │ │ │  │
│  │  │  │                    │ SectionView  │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 12: HotelUpsell                                                            │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ HotelUpsell  │───▶│ HotelUpsell  │───▶│ HotelUpsell  │                              │ │ │  │
│  │  │  │  │ Model        │    │ ViewModel    │    │ View         │                              │ │ │  │
│  │  │  │  │ WidgetFilter │    │              │    │              │                              │ │ │  │
│  │  │  │  │ Model        │    │              │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │ │ │  │
│  │  │  │                    │ HotelUpsell  │    │ HotelUpsell  │    │ WidgetFilter │          │ │ │  │
│  │  │  │                    │ ItemView     │    │ ItemViewModel │    │ View         │          │ │ │  │
│  │  │  │                    │              │    │              │    │              │          │ │ │  │
│  │  │  │                    └──────────────┘    └──────────────┘    └──────────────┘          │ │ │  │
│  │  │  │                                                                                          │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ WidgetFilter │                                                     │ │ │  │
│  │  │  │                    │ VM            │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 13: NewsAndTravel                                                         │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ NewsAndTravel│───▶│ NewsAndTravel│───▶│ NewsAndTravel│                              │ │ │  │
│  │  │  │  │ Model        │    │ Widget        │    │ SectionView  │                              │ │ │  │
│  │  │  │  │              │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ NewsAndTravel│                                                     │ │ │  │
│  │  │  │                    │ View         │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 14: SecondaryFeatureGrid                                                    │ │ │  │
│  │  │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                              │ │ │  │
│  │  │  │  │ Secondary    │───▶│ Secondary    │───▶│ Secondary    │                              │ │ │  │
│  │  │  │  │ Feature       │    │ Feature      │    │ Feature      │                              │ │ │  │
│  │  │  │  │ Models        │    │ Grid         │    │ GridView     │                              │ │ │  │
│  │  │  │  │              │    │ ViewModel    │    │              │                              │ │ │  │
│  │  │  │  └──────────────┘    └──────────────┘    └──────────────┘                              │ │ │  │
│  │  │  │         │                    │                  │                                         │ │ │  │
│  │  │  │         └────────────────────┼──────────────────┘                                         │ │ │  │
│  │  │  │                             ▼                                                           │ │ │  │
│  │  │  │                    ┌──────────────┐                                                     │ │ │  │
│  │  │  │                    │ FeatureItem  │                                                     │ │ │  │
│  │  │  │                    │ View         │                                                     │ │ │  │
│  │  │  │                    └──────────────┘                                                     │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widget Type 15: HeroWidget                                                             │ │ │  │
│  │  │  │  ┌──────────────┐                                                                      │ │ │  │
│  │  │  │  │ IxiMediaType │                                                                      │ │ │  │
│  │  │  │  │ (Enum)       │                                                                      │ │ │  │
│  │  │  │  └──────────────┘                                                                      │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsHomeWidgetViewModel (Enum - Widget Registry)                                  │ │ │  │
│  │  │  │  • case partnerOffers(ImageCarouselConfig)                                            │ │ │  │
│  │  │  │  • case exclusivePartnerOffers(ImageCarouselConfig)                                   │ │ │  │
│  │  │  │  • var id: String (Identifiable conformance)                                          │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 5: UTILITIES LAYER                                                │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Utilities/                                                                                   │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsAnalytics (Analytics Wrapper)                                                 │ │ │  │
│  │  │  │  • analyticsProvider: FlightsAnalyticsProviderDelegate? (static)                      │ │ │  │
│  │  │  │  • sendCleverTapEvent(_:properties:) → Bool                                           │ │ │  │
│  │  │  │  • logError(_:properties:function:file:line:)                                         │ │ │  │
│  │  │  │  • logWarning(_:properties:function:file:line:)                                        │ │ │  │
│  │  │  │  • logInfo(_:properties:function:file:line:)                                          │ │ │  │
│  │  │  │  • logDebug(_:properties:function:file:line:)                                          │ │ │  │
│  │  │  │  • Delegates to host app's analytics provider                                          │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsAnalyticsEvents (Event Constants)                                             │ │ │  │
│  │  │  │  • Predefined event names and constants                                                │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  AssetCache (Image Caching System)                                                    │ │ │  │
│  │  │  │  • AssetCache.swift - Main cache interface                                            │ │ │  │
│  │  │  │  • AssetCache+Compat.swift - Compatibility layer                                      │ │ │  │
│  │  │  │  • AssetCache+Sync.swift - Synchronous operations                                     │ │ │  │
│  │  │  │  • AssetCacheConfig.swift - Configuration                                             │ │ │  │
│  │  │  │  • AssetPathUtility.swift - Path utilities                                            │ │ │  │
│  │  │  │  • AssetStorage.swift - Storage implementation                                         │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsAssets (Asset Management)                                                    │ │ │  │
│  │  │  │  • Asset bundle access                                                                │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Extensions/                                                                           │ │ │  │
│  │  │  │  • ArrayExtension.swift                                                                │ │ │  │
│  │  │  │  • CenteredBehavior.swift                                                               │ │ │  │
│  │  │  │  • Collection+SafeSubscript.swift                                                      │ │ │  │
│  │  │  │  • ComponentExtension.swift                                                             │ │ │  │
│  │  │  │  • StringExtension.swift                                                                │ │ │  │
│  │  │  │  • UIScreen+Extras.swift                                                                │ │ │  │
│  │  │  │  • UIVIewExtension.swift                                                                │ │ │  │
│  │  │  │  • ViewExtension.swift                                                                  │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Helpers/                                                                              │ │ │  │
│  │  │  │  • IxiImageView.swift - Image loading helper                                          │ │ │  │
│  │  │  │  • IxiMediaView/                                                                      │ │ │  │
│  │  │  │    - IxiMediaView.swift - Media display component                                      │ │ │  │
│  │  │  │    - IxiMediaViewModel.swift - Media view model                                        │ │ │  │
│  │  │  │    - VideoPlayerContainerView.swift - Video player                                     │ │ │  │
│  │  │  │  • ShimmerAnimationView.swift - Loading shimmer effect                                  │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Modifier/                                                                             │ │ │  │
│  │  │  │  • BecomingVisibleModifier.swift - View visibility tracking                           │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  NetworkMockingExamples.swift - Network mocking utilities                              │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 6: DEEP LINKING LAYER                                             │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  DeepLinking/                                                                              │ │  │
│  │  │                                                                                             │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsDeepLinkRouter (Singleton)                                                    │ │ │  │
│  │  │  │  • shared: Static singleton instance                                                  │ │ │  │
│  │  │  │  • interceptor: FlightsExternalDeepLinkInterceptor? (weak)                            │ │ │  │
│  │  │  │  • navigationController: UINavigationController? (weak)                             │ │ │  │
│  │  │  │  • parser: FlightsDeepLinkParser (private)                                           │ │ │  │
│  │  │  │                                                                                         │ │ │  │
│  │  │  │  • open(url:from:) - Entry point for deeplink handling                                │ │ │  │
│  │  │  │    - Tries native Flights routes first                                                │ │ │  │
│  │  │  │    - Delegates to host app interceptor if available                                   │ │ │  │
│  │  │  │    - Falls back to default handling                                                    │ │ │  │
│  │  │  │                                                                                         │ │ │  │
│  │  │  │  • handleFlightsNative(_:from:) - Internal routing                                    │ │ │  │
│  │  │  │    - Handles Flights SDK native URLs                                                   │ │ │  │
│  │  │  │    - Routes: .flightsHome, .trackFlights, .exploreDestination, .unknown               │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsDeepLinkParser                                                                  │ │ │  │
│  │  │  │  • parse(_ url: URL) → FlightsDeepLinkRoute                                           │ │ │  │
│  │  │  │  • Parses URL and returns route enum                                                  │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  FlightsDeepLinkRoute (Enum)                                                           │ │ │  │
│  │  │  │  • case flightsHome                                                                    │ │ │  │
│  │  │  │  • case trackFlights                                                                   │ │ │  │
│  │  │  │  • case exploreDestination                                                              │ │ │  │
│  │  │  │  • case unknown                                                                         │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 7: VIEWS LAYER (Common & Widget Views)                            │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Views/                                                                                      │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Common/                                                                                │ │ │  │
│  │  │  │  • HeadingsSubheadings.swift - Reusable heading components                             │ │ │  │
│  │  │  │  • IxiBackdropBlur.swift - Backdrop blur effect                                        │ │ │  │
│  │  │  │  • IxiHeadingModels.swift - Heading data models                                        │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Widgets/                                                                              │ │ │  │
│  │  │  │  • ImageCarousel/                                                                       │ │ │  │
│  │  │  │    - Models/ImageCarouselModel.swift                                                    │ │ │  │
│  │  │  │    - Views/ImageCarouselSectionView.swift                                               │ │ │  │
│  │  │  │    - Views/ImageCarouselItemView.swift                                                  │ │ │  │
│  │  │  │  • RecentSearches/                                                                      │ │ │  │
│  │  │  │    - Models/RecentSearchModel.swift                                                    │ │ │  │
│  │  │  │    - Views/RecentSearchesSectionView.swift                                             │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                              LAYER 8: RESOURCES LAYER                                                │  │
│  │  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐ │  │
│  │  │  Resources/                                                                                  │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Assets.xcassets/                                                                      │ │ │  │
│  │  │  │  • airplaneSeat.imageset/                                                               │ │ │  │
│  │  │  │  • ArrowrightHeader.imageset/                                                            │ │ │  │
│  │  │  │  • exploreBackground.imageset/                                                           │ │ │  │
│  │  │  │  • Flight Tilted Filled.imageset/                                                        │ │ │  │
│  │  │  │  • flightsHomeBackground.imageset/                                                       │ │ │  │
│  │  │  │  • flightTiltedFilled.imageset/                                                          │ │ │  │
│  │  │  │  • homeFailurePlaceholder.imageset/                                                       │ │ │  │
│  │  │  │  • Star21.imageset/                                                                      │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  Lottie/                                                                                │ │ │  │
│  │  │  │  • Walk.lottie - Animation file                                                         │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  │                                                                                               │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────────────────────────────────┐ │ │  │
│  │  │  │  MockDataJson/                                                                         │ │ │  │
│  │  │  │  • whybook_mock.json - Mock data for testing                                            │ │ │  │
│  │  │  └────────────────────────────────────────────────────────────────────────────────────────┘ │ │  │
│  │  └──────────────────────────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                    │
                                                    │ External Dependencies
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              EXTERNAL DEPENDENCIES                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐                  │
│  │ ixigoNetworkSDK  │  │ ixigoUI         │  │ FlightsTheme    │  │ dotlottie-ios    │                  │
│  │ (Network Layer)  │  │ (UI Components) │  │ (Theme System)  │  │ (Animations)      │                  │
│  │                  │  │                 │  │                 │  │                  │                  │
│  │ NetworkManagerV2 │  │ IxigoUI         │  │ Theme           │  │ Lottie           │                  │
│  │ • Shared         │  │ .configure()    │  │ Configuration   │  │ Animation        │                  │
│  │   instance       │  │                 │  │                 │  │                  │                  │
│  │ • Protocol-based │  │                 │  │                 │  │                  │                  │
│  │   headers        │  │                 │  │                 │  │                  │                  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘  └──────────────────┘                  │
│                                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Internal Data Flow Patterns

### 1. SDK Initialization Flow
```
Host App calls:
  ixigoFlightsSDK.shared.configure(config)
    │
    ├─▶ Stores config (protocol delegates)
    ├─▶ Sets FlightsAnalytics.analyticsProvider
    ├─▶ Sets NetworkExtraHeaders.provider
    ├─▶ Sets FlightsDeepLinkRouter.shared.interceptor
    └─▶ Configures IxigoUI theme
```

### 2. Home View Data Flow
```
FlightsHomeView created
    │
    ├─▶ Creates FlightsHomeViewModel (@StateObject)
    │       │
    │       ├─▶ init() triggers:
    │       │     │
    │       │     ├─▶ loadWhyBookData()
    │       │     │     │
    │       │     │     └─▶ FlightHomeClient.fetchWhyBook()
    │       │     │           │
    │       │     │           └─▶ NetworkManagerV2.request()
    │       │     │                 │
    │       │     │                 └─▶ API Server
    │       │     │                       │
    │       │     │                       └─▶ Response → whyBookData (@Published)
    │       │     │
    │       │     └─▶ loadWidgets() (async)
    │       │           │
    │       │           └─▶ FlightsHomeWidgetsService.fetchWidgets()
    │       │                 │
    │       │                 └─▶ FlightHomeClient.fetchHomeWidgets()
    │       │                       │
    │       │                       └─▶ NetworkManagerV2.request()
    │       │                             │
    │       │                             └─▶ API Server
    │       │                                   │
    │       │                                   └─▶ Response → DTOs
    │       │                                         │
    │       │                                         └─▶ Maps to ViewModels
    │       │                                               │
    │       │                                               └─▶ widgets (@Published)
    │       │
    │       └─▶ View observes @Published properties
    │             │
    │             └─▶ SwiftUI automatically re-renders
```

### 3. Widget Data Flow (Example: ExploreMasonrySection)
```
ExploreMasonrySectionView created
    │
    ├─▶ Creates ExploreMasonryViewModel (@StateObject)
    │       │
    │       ├─▶ init() triggers:
    │       │     │
    │       │     └─▶ ExploreMasonryService.fetchDestinations()
    │       │           │
    │       │           └─▶ NetworkManagerV2.request()
    │       │                 │
    │       │                 └─▶ API Server
    │       │                       │
    │       │                       └─▶ Response → destinations (@Published)
    │       │
    │       └─▶ View observes @Published properties
    │             │
    │             └─▶ SwiftUI renders MasonryGrid
```

### 4. Analytics Event Flow
```
SDK Component (e.g., Widget)
    │
    ├─▶ FlightsAnalytics.sendCleverTapEvent(...)
    │       │
    │       └─▶ Checks analyticsProvider (from config)
    │             │
    │             └─▶ Delegates to host app's analytics provider
    │                   │
    │                   └─▶ Host app's AnalyticsCenter
```

### 5. Deep Link Flow
```
Widget/Component calls:
  FlightsDeepLinkRouter.shared.open(url, from: vc)
    │
    ├─▶ FlightsDeepLinkParser.parse(url)
    │       │
    │       └─▶ Returns FlightsDeepLinkRoute enum
    │
    ├─▶ handleFlightsNative(url, from: vc)
    │       │
    │       ├─▶ If .flightsHome → Push FlightsHomeHostingViewController
    │       ├─▶ If .trackFlights → Handle tracking flow
    │       └─▶ If .exploreDestination → Handle explore flow
    │
    ├─▶ If not handled, delegate to interceptor
    │       │
    │       └─▶ Host app's FlightsExternalDeepLinkInterceptor
    │             │
    │             └─▶ Host app handles external URLs
    │
    └─▶ Fallback: Default handling
```

---

## Component Relationships

### 1. ViewModel → View Relationship
- **@StateObject**: View owns and creates ViewModel
- **@Published**: ViewModel properties trigger view updates
- **@MainActor**: Ensures UI updates on main thread

### 2. Service → ViewModel Relationship
- Services are injected via initializer or created internally
- Services handle async operations (API calls)
- ViewModels observe service results and update @Published properties

### 3. Client → Service Relationship
- Clients are low-level API callers
- Services orchestrate multiple client calls
- Services map DTOs to ViewModels

### 4. Protocol Delegate Pattern
- SDK defines protocols (interfaces)
- Host app implements protocols
- SDK uses protocols without knowing implementation details
- Enables dependency inversion

### 5. Widget Registry Pattern
- `FlightsHomeWidgetViewModel` enum acts as registry
- Each widget type is a case with associated ViewModel
- View switches on enum to render appropriate widget

---

## Key Design Patterns

### 1. Singleton Pattern
- `ixigoFlightsSDK.shared` - SDK entry point
- `FlightsDeepLinkRouter.shared` - Deep link routing
- `NetworkManagerV2.shared` - Network layer (from NetworkSDK)

### 2. Protocol-Based Dependency Injection
- SDK defines protocols for host app services
- Host app implements protocols
- SDK remains decoupled from host app implementation

### 3. MVVM Architecture
- **Model**: Decodable data structures (DTOs)
- **ViewModel**: Business logic, state management (@Published)
- **View**: SwiftUI views that observe ViewModels

### 4. Reactive Programming
- SwiftUI @Published properties
- Automatic view updates on state changes
- @MainActor ensures thread safety

### 5. Enum-Based Type Safety
- `FlightsHomeWidgetViewModel` enum for widget registry
- `FlightsDeepLinkRoute` enum for routing
- Type-safe widget handling

### 6. Service Layer Pattern
- Services orchestrate business logic
- Clients handle low-level API calls
- Clear separation of concerns

---

## SDK Package Structure

```
Packages/ixigoFlightsSDK/
├── Package.swift (SPM manifest)
├── README.md
└── Sources/ixigoFlightsSDK/
    ├── ixigoFlightsSDK.swift (Core)
    ├── Protocols/
    │   └── FlightsSDKIntegrationProtocols.swift
    ├── Features/
    │   └── Home/
    │       ├── FlightsHomeHostingViewController.swift
    │       ├── FlightsHomeView.swift
    │       ├── FlightsHomeViewModel.swift
    │       ├── FlightsHomeSearchForm.swift
    │       ├── FlightHomeClient.swift
    │       ├── FlightsHomeWidgetsService.swift
    │       └── Widgets/ (15 widget types)
    ├── DeepLinking/
    │   ├── FlightsDeepLinkRouter.swift
    │   ├── FlightsDeepLinkParser.swift
    │   └── FlightsDeepLinkRoute.swift
    ├── Utilities/
    │   ├── FlightsAnalytics.swift
    │   ├── FlightsAnalyticsEvents.swift
    │   ├── FlightsAssets.swift
    │   ├── AssetCache/
    │   ├── Extensions/
    │   ├── Helpers/
    │   └── Modifier/
    ├── Views/
    │   ├── Common/
    │   └── Widgets/
    └── Resources/
        ├── Assets.xcassets/
        ├── Lottie/
        └── MockDataJson/
```

---

## Dependencies

### Internal Dependencies
- **ixigoNetworkSDK**: Network layer (NetworkManagerV2)
- **ixigoUI**: UI components and design system
- **FlightsTheme**: Theme configuration
- **dotlottie-ios**: Lottie animation support

### External Dependencies (via Host App)
- **Auth Service**: Via FlightsAuthDelegate
- **Remote Config**: Via FlightsRemoteConfigDelegate
- **Analytics**: Via FlightsAnalyticsProviderDelegate
- **Network Headers**: Via FlightsNetworkHeaderDelegate
- **Deep Linking**: Via FlightsExternalDeepLinkInterceptor

---

## Thread Safety & Concurrency

### Main Thread Operations
- All ViewModels marked with `@MainActor`
- UI updates guaranteed on main thread
- @Published properties trigger on main thread

### Background Operations
- API calls happen on background threads (NetworkManagerV2)
- JSON decoding on background threads
- ViewModel updates switch to main thread via `MainActor.run`

### Async/Await Pattern
- Services use `async/await` for API calls
- `withCheckedContinuation` bridges completion handlers
- Task-based concurrency

---

## State Management

### ViewModel State
- `@Published` properties for reactive updates
- `ObservableObject` protocol conformance
- State changes trigger SwiftUI view updates

### Widget Registry State
- `FlightsHomeWidgetViewModel` enum array
- Managed by `FlightsHomeViewModel`
- Updated via `FlightsHomeWidgetsService`

### Auth State
- `FlightsAuthCenter.isLoggedIn` (computed property)
- NotificationCenter notifications for state changes
- Host app notifies SDK of auth changes

### Remote Config State
- `FlightsRemoteConfigCenter.notifyChanges()`
- NotificationCenter notifications
- Host app notifies SDK of config changes

---

*Document focused exclusively on Flights SDK internal architecture*


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

