# LEAG

**League Excellence & Growth** - A comprehensive multi-tenant SaaS platform for youth sports organizations.

## Overview

LEAG is an all-in-one platform designed for AAU teams, travel leagues, recreational programs, and athletic academies. It streamlines the complex operations of running a sports organization into one beautiful, intuitive dashboard.

## Features

- **Team Management** - Track rosters, eligibility, and team information in one centralized location
- **Smart Scheduling** - Auto-generate balanced schedules and bracket tournaments with conflict detection
- **Financial Tools** - Collect league fees, process payments via Stripe, and manage expenses seamlessly
- **Live Stats & Standings** - Real-time score updates, player statistics, and automated standings calculations
- **Event Registration** - Handle tournament sign-ups, tryouts, and camp registrations
- **Communication Hub** - Keep admins, coaches, players, and parents connected

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | Next.js 14+ (App Router), React, TypeScript |
| **Styling** | Tailwind CSS, shadcn/ui |
| **Backend** | Django 5.0+, Django REST Framework |
| **CMS** | Wagtail 7.x |
| **Database** | PostgreSQL 14+ |
| **Payments** | Stripe Connect |
| **Deployment** | Vercel (frontend), Railway (backend) |

## Platform Architecture

LEAG operates as a multi-tenant platform where each sports organization receives their own customized instance with:

- Custom branding (colors, logos, domain)
- Isolated data and member management
- Organization-specific feature configuration
- Integrated payment processing via Stripe Connect

## Mobile App (Coming Soon)

The LEAG mobile app will keep everyone connected on the go:
- Push notifications for game times and updates
- Live score tracking
- Team management from anywhere
- Available on iOS and Android

## Contact

- **Email**: contact@leag.app
- **Website**: [leag.app](https://leag.app)

## License

Proprietary - All rights reserved by Stryder Labs LLC
