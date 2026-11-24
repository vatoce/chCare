# ADR-0003: Mobile apps with React Native + Expo

- **Status:** Accepted
- **Date:** 2026-11-24
- **Supersedes:** PWA-first mobile plan in PRD v0.1

## Context
The mobile app is a primary working tool for servants (field visitation, QR check-in, often poor connectivity) and for parents (follow children remotely). It must run on iOS and Android, be reliable and fast on low-end Android devices, work offline, support push notifications, camera, and device biometrics for app unlock.

## Decision
- **React Native with Expo** (development builds, EAS Build, EAS Update for OTA), TypeScript, Expo Router.
- One codebase, role-based experience (servant, leader, parent, served person). Separate store listings via app variants can be added later.
- Offline-first: encrypted SQLite local database with a sync layer. Candidates: **WatermelonDB** (default) vs **PowerSync**; final pick made by a time-boxed spike in Phase 0 (criteria: conflict handling, Postgres/RLS compatibility, Arabic text, encrypted storage, bundle size).
- Secure storage: expo-secure-store for tokens; SQLCipher for local DB; screenshot protection on sensitive screens.
- Native modules are allowed through Expo config plugins / dev builds (no "Expo Go only" constraint).

## Why not Flutter?
Flutter is a strong option, but it would split the codebase into a second language (Dart), preventing sharing of Zod schemas, API client and rules types, and doubling the skill set needed by volunteers. 

**Revisit triggers:** sustained performance problems on target low-end devices, or a required attendance/device SDK that cannot be bridged in React Native.

## Why not PWA only?
Reliable background sync, push on iOS, camera/QR performance, secure local storage and store presence for parents are materially better in a native app. A web portal remains as a secondary channel.

## Consequences
- (+) Single TypeScript codebase for iOS and Android, shared packages, OTA fixes.
- (-) Store review and privacy-label work (Kids/Families policies) must be planned from Phase 1.
- (-) Offline sync is the highest-risk component; it gets tests and a dedicated spike.
