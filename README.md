# ACT Study Dashboard

A comprehensive ACT prep dashboard with user authentication, cloud data sync, and interactive study tools. Built as a single-page React application with Supabase backend.

## Features

- **5 ACT Sections** — English, Math, Reading, Science (optional), Writing (optional)
- **Flashcards** — 3D flip cards covering key concepts for each section
- **Practice Quizzes** — Timed quizzes with explanations for every answer
- **Full Exam Simulator** — Fullscreen timed test with section-by-section scoring and review
- **Focus Areas** — Weakness tracking based on exam results with personalized study recommendations
- **Study Guide** — Quick-reference guide for all ACT sections
- **Study Buddy Chatbot** — Pattern-matching chatbot with math evaluation (via mathjs)
- **User Authentication** — Email/password signup and login via Supabase Auth
- **Cloud Sync** — Exam history saved to Supabase database (replaces localStorage)
- **User Profiles** — First/last name, test date countdown, email/password management
- **Single-Session Enforcement** — Only one active login per account at a time

## Tech Stack

- **Frontend:** React 18 (CDN), Tailwind CSS (CDN), Babel (in-browser JSX), mathjs
- **Backend:** Supabase (Authentication + PostgreSQL database)
- **Hosting:** Vercel (static deployment)
- **Architecture:** Single HTML file with client-side rendering

## Setup

### 1. Supabase Project

Create a free project at [supabase.com](https://supabase.com) and run this SQL in the SQL Editor:

```sql
CREATE TABLE user_progress (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  progress_data JSONB DEFAULT '{}'::jsonb,
  session_token TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE user_progress ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_progress ADD CONSTRAINT user_progress_user_id_unique UNIQUE (user_id);

CREATE POLICY "Users can read own data"
  ON user_progress FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own data"
  ON user_progress FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own data"
  ON user_progress FOR UPDATE
  USING (auth.uid() = user_id);
```

### 2. Configure API Keys

Open `index.html` and replace the placeholder values near the top (lines 27-28):

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL_HERE';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY_HERE';
```

Find your keys in the Supabase dashboard: **Project Settings → API**.

### 3. Deploy to Vercel

```bash
npm install -g vercel
vercel login
vercel --prod
```

### 4. Configure Supabase Redirect URLs

In your Supabase dashboard, go to **Authentication → URL Configuration** and add your Vercel deployment URL to the Redirect URLs list.

## Local Development

Open `index.html` directly in a browser, or run a local server:

```bash
npx serve .
```

## Project Structure

```
├── index.html      ← Entire application (single file)
├── package.json    ← Dependencies and metadata
├── .gitignore      ← Git ignore rules
└── README.md       ← This file
```

## Database Schema

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| user_id | UUID | References auth.users, unique constraint |
| progress_data | JSONB | Stores exam history and user progress |
| session_token | TEXT | Single-session enforcement token |
| created_at | TIMESTAMPTZ | Row creation time |
| updated_at | TIMESTAMPTZ | Last update time |

## Security

- **Row Level Security (RLS):** Users can only access their own data
- **Single-session enforcement:** New logins invalidate previous sessions
- **Anon key is safe for client-side use** — security is enforced via RLS policies on the database
