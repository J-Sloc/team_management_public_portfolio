# High School Sports Team Management Platform

## Overview

A comprehensive team management system designed for high school athletic departments. This platform streamlines operations for athletic directors, coaches, and student-athletes by centralizing roster management, academic and health tracking, performance analytics, and event scheduling.

**Designed for:** Athletic Directors | Head Coaches | Student-Athletes

---

## Problem Solved

High school athletic programs juggle complex workflows across multiple spreadsheets and systems:
- Coaches manually track athlete eligibility, GPA, and medical status
- Athletic directors lack visibility into cross-team compliance and performance trends
- Student-athletes have fragmented access to their own records and schedules
- Data consistency breaks during roster imports and seasonal transitions

This platform unifies these workflows into a single source of truth with intelligent insights and role-specific interfaces.

---

## Key Features

### Core Functionality
- **Unified Roster Management** – Athlete profiles with academic standing, medical status, eligibility years, and sport-specific metrics
- **Academic & Eligibility Tracking** – Real-time GPA, term compliance, and eligibility projections with AI-powered risk flagging
- **Health & Medical Records** – Injury tracking, rehab sessions, appointment attendance, and medical clearance status
- **Event & Calendar Management** – Team schedules, games, practices, medical appointments, and academic deadlines in one view
- **Performance Analytics** – ML augmented deterministic analytics snapshots for workouts, rankings, and athlete trend analysis

### Intelligence & Assistance
- **Role-Based AI Assistant** – Contextual, safe AI-generated insights answering questions about team performance, at-risk athletes, and compliance issues
- **Analytics Snapshots** – Pre-computed performance summaries and trend projections based on persisted data (not real-time ML)
- **Smart Filtering** – Advanced filtering and search across academic, medical, and performance data

### Athlete Portal
- **Personal Dashboard** – Academic summaries, medical status, upcoming events, workout data, and personal rankings
- **Self-Service Records** – Athletes can view their own academics, health records, and eligibility status

### Administrator Features
- **Reviewed CSV Import** – Safe bulk roster uploads with preview, exact-match deduplication, and all-or-nothing apply logic
- **Team-Scoped Access Control** – Role-based permissions ensuring coaches see only their team, ADs see all teams
- **Multi-Season Support** – Institution and season-level organization for managing historical data

---

## Technical Architecture

### Tech Stack
- **Frontend:** Next.js 16 (App Router), TypeScript, React 19, Tailwind CSS
- **Backend:** Next.js API Routes, TypeScript
- **Database:** PostgreSQL (Neon Serverless), Prisma ORM
- **Authentication:** NextAuth.js with email/password credentials
- **AI Integration:** OpenAI API (optional; deterministic fallback available)
- **State Management:** React Query v5
- **Testing:** Vitest

### Architecture Patterns

#### Role-Based Access Control (RBAC)
- Three-tier permission model: Athletic Director (institution-level), Coach (team-level), Athlete (self-service)
- Server-side route guards and API middleware enforce scoping
- Shared access helpers prevent data leaks across team boundaries

#### Analytics Snapshot Layer
- **Why:** Avoids expensive real-time ML calls and latency issues
- **How:** Deterministic analytics computations are run on-demand or scheduled, persisted to the database
- **What:** Snapshots include training readiness, adherence metrics, performance trends, and eligibility projections
- **Result:** Instant data access without external dependencies

#### Safe LLM/Assistant Architecture
- Contextualization is deterministic and server-side: assistant builds context from persisted snapshots, academic/health records, and upcoming events
- LLM receives only aggregated, non-sensitive summaries
- Deterministic fallback responses available when API keys are not configured
- No direct data streaming to external services

#### ML Export Separation
- Athlete data export hooks are isolated from the main app workflow
- Exports serve as the contract boundary between the app and ML training pipelines
- The app owns data validation and access control; ML tooling operates independently downstream

### Data Model
Core entities (examples): `User`, `Team`, `Athlete`, `AcademicRecord`, `HealthRecord`, `Event`, `Note`, `WorkoutInstance`, `EventRanking`, `ImportJob`, and analytics snapshots.

Multi-team support and flexible JSON storage enable sport-specific extensions without schema changes.

---

## Use Cases

### Athletic Director
- Overview dashboard with institution-wide KPIs (team GPA averages, ineligibility count, compliance status)
- AI-generated insights flagging at-risk athletes across all teams
- Bulk roster import with safe review workflow
- Access to all team data, analytics, and import history

### Head Coach and Assistant Coaches
- Team dashboard with roster, academic status, medical clearances, and upcoming events
- Athlete detail pages with academic, health, and performance context
- Quick action queue for high-priority athlete reviews
- Local CSV import for roster updates

### Student-Athlete
- Personal dashboard with academic standing, medical status, and eligibility summary
- Calendar with personal events and team schedules
- Workout logs, rankings, and performance summaries
- Self-service access to their own records

---

## Development Status

**Current Phase:** Core coaching workflows and athlete portal implemented. Analytics snapshot computation and AI assistant operational. Reviewed CSV import foundation established for scalable data onboarding.

**Recent:** Import job persistence, deterministic roster matching, duplicate detection, and safe apply/report flow added.

---

## Deployment & Scaling

- Hosted on Vercel (Next.js-optimized deployment)
- PostgreSQL via Neon (serverless, auto-scaling)
- Environment-based configuration for dev/staging/production
- Test suite (Vitest) validates core logic before deployment

---

## Example Roadmap Features

- Multi-sport institution support
- Recruiting module for college-bound athletes
- Nutrition and strength & conditioning integrations
- Parent/guardian access tier
- Advanced ML model training on exported datasets
- Fuzzy athlete matching and merge tools
- Comprehensive audit logs and compliance reporting

---

## Philosophy

This platform prioritizes **data integrity** and **safe automation**:
- Reviewed imports prevent bulk data errors
- Deterministic analytics avoid real-time ML latency and failure modes
- Role-based scoping ensures institutional compliance and privacy
- AI assistance is contextualized and deterministic rather than generative

The architecture is built to scale from single-team pilots to multi-sport institutions while maintaining clear data ownership and minimal external dependencies.
