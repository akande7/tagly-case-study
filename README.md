# Tagly: lost property management for venues

> **Case study.** Tagly's source code is private because it runs in production. This repo explains what Tagly does, how it's built and how it keeps venue data separate and secure. I'm happy to walk through the code in an interview.

Tagly is a lost property app for venues such as clubs, hotels and restaurants. It's live at **House of KOKO**, where **8 staff** use it to manage **100+ lost items a month**.

Staff photograph each item they find and give it the number on its physical tag. When the owner collects it, staff record who took it, with a signature.

## What it does

- **Log items:** photograph each item and record the number on its physical tag
- **Track items:** see everything a venue is holding and what has already gone home
- **Hand over items:** record the collector and capture their signature on screen
- **Built for the venue's iPads:** works in landscape and portrait, and on phones as a full-screen home-screen app
- **Multi-venue:** each venue has its own login and only ever sees its own items

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4 |
| Backend | Next.js server actions and API routes |
| Database, auth and storage | Supabase (Postgres, Auth, Storage) |
| Hosting | Vercel |
| Built with | Claude Code (implementation) and Claude Design (UI design) |

## Architecture

```
iPad / phone (PWA)
      │
      ▼
Next.js on Vercel ── every page, server action and API route checks the signed-in staff login
      │
      │  talks to Supabase as the signed-in user (never with the service role key)
      ▼
Supabase
 ├─ Postgres: Row Level Security on every table, scoped to the login's venue
 ├─ Auth: one login per venue, public sign-up disabled
 └─ Storage: private buckets for photos and signatures, served through short-lived signed links
```

## Security

Security shaped the design from the start:

- **Venue isolation:** the venue always comes from the signed-in login, never from the request. Row Level Security limits every table and storage bucket to that venue.
- **Least privilege:** the app never uses the Supabase service role key. It's only used by admin scripts and is never deployed.
- **Private files:** photos are served through signed links that expire after 1 hour, and signatures after 5 minutes.
- **Upload validation:** photos must be JPEG, checked by their first bytes rather than the file extension, and under 4 MB. Signatures must be PNG and under 1 MB. The storage buckets enforce the same rules.
- **Rate limiting:** on top of Supabase's own limits, Tagly allows 5 failed sign-ins per email per IP address, and 20 per IP address, in 15 minutes.
- **Controlled accounts:** public sign-up is turned off, and venue logins are only created through an admin script.

## Testing

- **Automated venue-separation test:** it creates two temporary venues and tries every way of reaching one venue's items and files from the other, both directly through Supabase and through the running app. It also checks that signed-out requests are refused, that upload rules hold and that signed links expire. It runs after any change to data access.
- **Type checks, linting and a production build** are part of the standard checks.
- **Safe test data:** the seed script refuses to run against real venues or in production.

## Building with AI

I built Tagly with Claude Code for implementation and Claude Design for the interface. The automated separation tests above check that the generated code keeps every venue's data isolated, and they run after every change to data access.
