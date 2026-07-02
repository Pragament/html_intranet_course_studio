# EduFlow — Teacher & Student Learning Platform

EduFlow is a dual‑role web application that empowers teachers to create and manage online courses, while students can browse, take courses, and track their progress. Built with **Supabase** for authentication and storage, **Tailwind CSS** for styling, and vanilla JavaScript, it’s a fully functional single‑page app that runs in any modern browser.

---

## 🚀 Features

### For Teachers
- **Google OAuth login** – secure authentication with your Google account.
- **Course management** – create, edit, and delete courses with titles, descriptions, and cover images.
- **Topic editor** – add/remove/edit topics with rich content and images (stored in Supabase Storage).
- **Practice questions** – build multiple‑choice questions with instant‑feedback reasoning.
- **AI‑powered CSV import** – upload a CSV file to auto‑generate topics and questions (great for bulk creation).
- **Student progress dashboard** – view each student’s completion percentage and topic progress per course.

### For Students
- **Searchable course catalog** – filter by title/description and sort by newest, oldest, or title.
- **Personal progress tracking** – your completion status and answers are saved per course.
- **Resume or restart** – continue from where you left off or reset your progress entirely.
- **Jump between topics** – navigate freely through course content.
- **Practice questions with instant feedback** – see correct/incorrect indicators and detailed reasoning after answering.

---

## 🧰 Prerequisites

- A **Supabase** account (free tier works fine).
- A **Google Cloud** project (to enable Google OAuth in Supabase).
- Any modern web browser (Chrome, Firefox, Edge, etc.) – no build tools required.

---

## ⚙️ Setup

### 1. Clone or download this repository

```bash
git clone https://github.com/yourusername/eduflow.git
cd eduflow
```

Alternatively, just save the provided `index.html` file into a new folder.

### 2. Configure Supabase

1. Create a new Supabase project at [supabase.com](https://supabase.com).
2. In your project dashboard, go to **Authentication** → **Providers** and enable **Google**.
   - Copy your **Google Client ID** and **Secret** from the Google Cloud Console and add them to Supabase.
3. In **Storage**, create a new bucket named `course-images` (public) to store topic images.
4. In **SQL Editor**, run the following SQL to create the required tables:

```sql
-- Courses table
CREATE TABLE courses (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  description TEXT,
  image_url TEXT,
  teacher_id TEXT NOT NULL,
  topics JSONB DEFAULT '[]'::jsonb,
  practice_questions JSONB DEFAULT '[]'::jsonb,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Student progress table
CREATE TABLE student_progress (
  id BIGSERIAL PRIMARY KEY,
  student_id TEXT NOT NULL,
  course_id TEXT NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  current_topic INT DEFAULT 0,
  completed_topics JSONB DEFAULT '[]'::jsonb,
  started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_accessed TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Student answers for practice questions
CREATE TABLE practice_answers (
  id BIGSERIAL PRIMARY KEY,
  student_id TEXT NOT NULL,
  course_id TEXT NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  question_id TEXT NOT NULL,
  selected_answer TEXT NOT NULL,
  attempted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security (optional, add your own policies)
ALTER TABLE courses ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_progress ENABLE ROW LEVEL SECURITY;
ALTER TABLE practice_answers ENABLE ROW LEVEL SECURITY;
```

### 3. Set environment variables

In the `index.html` file, locate the Supabase configuration section (near the top of the JavaScript) and replace the placeholders:

```javascript
const SUPABASE_URL = 'https://your-project-url.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';
```

> **Tip**: You can find these values in your Supabase project dashboard under **Settings** → **API**.

---

## 📦 Database Schema Details

| Table | Purpose |
|-------|---------|
| `courses` | Stores all courses. The `topics` and `practice_questions` fields are JSON arrays containing the full structure of topics and questions. |
| `student_progress` | Tracks each student’s current topic index and the list of completed topic indices (0‑based). |
| `practice_answers` | Saves every attempt for each practice question, including the selected answer and timestamp. |

---

## 🧪 Running the Application

Since it’s a static HTML file with CDN dependencies, you can:

- **Open directly** – double‑click `index.html` in your browser.
- **Use a local server** (recommended for OAuth redirects) – for example, with VS Code’s Live Server extension or Python’s `http.server`:

```bash
python -m http.server 8000
```

Then visit `http://localhost:8000`.

> **Note**: Google OAuth requires a valid redirect URI. In Supabase, set your allowed redirect URLs to include `http://localhost:8000` (or your production domain).

---

## 👩‍🏫 Usage Guide

### Teacher Workflow

1. **Sign in** with your Google account using the **“Sign in with Google”** button.
2. Click **“New Course”** to create a course. Fill in the title, description, and optional image URL.
3. In the course editor, you can:
   - **Add topics** – each topic can have text content and multiple images (upload from your device).
   - **Add practice questions** – enter four options, mark the correct one, and add a reasoning explanation.
   - **Import from CSV** – upload a CSV file with columns: `topic, content, question, option_a, option_b, option_c, option_d, correct_answer, reasoning`. The system will parse and populate topics and questions automatically.
4. **Save** the course – it becomes available for students immediately.
5. Switch to the **“Student Progress”** tab to see how each student is progressing through the course.

### Student Workflow

1. **Sign in** with Google (or use the mock mode – see below).
2. Browse the course catalog – use the search bar, filter by “My Courses”, and sort options.
3. Click **“Start Course”** or **“Resume”** to enter the player.
4. Navigate through topics using the **Prev** / **Next** buttons.
5. Mark a topic as **complete** (your progress is saved automatically).
6. Answer practice questions – after selecting an option, you’ll see instant feedback and reasoning.
7. Use **“Reset Progress”** to start the course from scratch.
8. Return to the dashboard anytime.

---

## 🧑‍💻 Development & Customization

- **Pure vanilla JS** – no frameworks, easy to modify.
- **Tailwind CSS** – utility‑first styling; you can swap in your own CSS.
- **Mock mode** – if Supabase keys are not set, the app falls back to localStorage, allowing offline testing.
- **Extend the CSV format** – modify the `handleCSVFile` function to accept additional columns (e.g., `topic_image`).
- **Add new features** – the code is structured with clear sections (auth, database, rendering, events). Look for the `// =====` comments.

### Adding Custom Styling

Replace the Tailwind classes in the HTML or include a custom `<style>` block. The app already uses responsive design.

### Modifying the Data Model

If you need additional fields (e.g., course duration), update the Supabase table schemas and adjust the `state.currentCourse` object accordingly.

---

## ☁️ Deployment

You can deploy this single HTML file to any static hosting service:

- **Vercel / Netlify** – drag & drop the folder.
- **GitHub Pages** – push the file to a repo and enable Pages.
- **Supabase** – you can even host it via their Edge Functions, but a static host is simpler.

Remember to update `SUPABASE_URL` and `SUPABASE_ANON_KEY` for your production environment, and set the correct OAuth redirect URIs in Supabase.

---

## 🛠️ Technologies Used

- [Supabase](https://supabase.com) – backend (Auth, Database, Storage)
- [Tailwind CSS](https://tailwindcss.com) – styling
- [Font Awesome](https://fontawesome.com) – icons
- Vanilla JavaScript – all logic is self‑contained in one file

---

## 📄 License

This project is open‑source.

---

## 🙏 Acknowledgements

- Built with ❤️ for educators and learners.
- Inspired by modern learning management systems.

---

**Start creating your courses today!** If you encounter any issues, please open an issue or contribute to the project.
