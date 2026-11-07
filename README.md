# TalentHub — Enterprise Campus Placement Management System

TalentHub is a production-grade campus recruitment and placement management platform designed to automate and streamline the college hiring season across students, corporate recruiters, and the university placement cell.

---

## 🎯 Solved Institutional Placement Policies (Silent Decisions)

This version resolves all undocumented assumptions, security vulnerabilities, and workflow shortcuts identified in initial baseline prototypes:

| Feature Dimension | Production Standard Enforced |
|---|---|
| **Placement Policy** | **Strict One-Offer & Dream Upgrade Policy**: Accepting an offer marks a student as `PLACED` and automatically withdraws other pending standard applications (`WITHDRAWN_PLACED`). Placed students may only apply for designated **Dream Tier** drives ($\ge 1.5\times$ current CTC). |
| **Recruitment Funnel** | **Backend State Machine Guards**: Offers cannot be issued unless a candidate has progressed through and passed all scheduled interview rounds (`PASS` outcome). |
| **Interview Model** | **Multi-Round Pipeline ($1:N$)**: Supports sequential rounds (Online Assessments, Technical Interviews, Managerial, HR) with 0–10 scorecards, interviewer notes, and audit history. |
| **Academic Eligibility** | **Multi-Factor Criteria**: Strict filtering on discipline/branch matching, minimum CGPA, and maximum active backlogs with dynamic eligibility checklist feedback. |
| **Relational Integrity** | **Foreign Key Enforcement**: `PRAGMA foreign_keys = ON;` and `PRAGMA journal_mode = WAL;` enabled on all database connections. Unique composite constraint `UNIQUE(student_id, job_id)`. |
| **Security & Auth** | **JWT with Database Verification**: Secrets managed via `.env`; `require_role` actively checks account existence and active status in the database on every authenticated request. |
| **User Interface** | **Executive Light Theme Portal**: Clean, modern corporate dashboard (inspired by Handshake and Linear) with high-contrast typography, interactive modals, and real-time status badges. |

> For a complete architectural audit of all silent decisions, consult [`SILENT_DECISIONS.md`](./SILENT_DECISIONS.md) or [`TalentHub_Silent_Decisions.docx`](./TalentHub_Silent_Decisions.docx).

---

## 🏗️ System Architecture

```
talenthub/
├── backend/
│   ├── app/
│   │   ├── api/          # Routers (auth, students, companies, jobs, applications, interviews, offers, coordinator)
│   │   ├── core/         # Config (pydantic-settings), Async SQLite (WAL & FK PRAGMA), Security (JWT)
│   │   ├── models/       # SQLAlchemy ORM models with constraints & relationships
│   │   ├── schemas/      # Pydantic request/response schemas
│   │   ├── services/     # PlacementService (state machine), EligibilityService (branch + dream logic)
│   │   └── seed.py       # Comprehensive initial seed data
│   ├── requirements.txt  # Python dependencies
│   ├── .env.example      # Environment variable template
│   ├── .env              # Active configuration
│   └── main.py           # FastAPI entry point with lifespan seeder
└── frontend/
    ├── src/
    │   ├── api/          # Axios client with 401 interceptor
    │   ├── components/   # Navbar, Modal, StatusBadge, ProtectedRoute
    │   ├── context/      # AuthContext for session management
    │   ├── pages/        # Login, Register, StudentDashboard, CompanyDashboard, CoordinatorDashboard
    │   ├── App.jsx       # React Router tree
    │   ├── index.css     # Clean light theme styles
    │   └── main.jsx
    ├── package.json
    ├── tailwind.config.js
    └── vite.config.js    # Vite build configuration
```

---

## 🚀 How to Run Locally

### 1. Prerequisites
- **Python 3.9+**
- **Node.js v18+**

### 2. Backend Setup (Terminal 1)

```bash
cd backend

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Start the FastAPI server
uvicorn main:app --reload --port 8000
```

- **Interactive API Documentation (Swagger):** `http://127.0.0.1:8000/docs`
- **Health Check:** `http://127.0.0.1:8000/health`

### 3. Frontend Setup (Terminal 2)

```bash
cd frontend

# Install dependencies
npm install

# Start the development server
npm run dev
```

- **Web Portal Access:** Open `http://localhost:5174` (or `http://localhost:5173`) in your browser.

---

## 👥 Seed Demo Accounts

All demo accounts use password: **`password123`** (available with 1-click on the sign-in screen).

| Persona | Name | Email | Details / Branch |
|---|---|---|---|
| **Student** | Rahul Sharma | `rahul@student.talenthub.edu` | Computer Science (CGPA: 8.4, 0 Backlogs) |
| **Student** | Priya Nair | `priya@student.talenthub.edu` | Information Technology (CGPA: 9.2, 0 Backlogs) |
| **Student** | Sneha Patil | `sneha@student.talenthub.edu` | Mechanical (CGPA: 7.9, 0 Backlogs) |
| **Student** | Aman Verma | `aman@student.talenthub.edu` | Electronics (CGPA: 6.8, 2 Backlogs) |
| **Company** | ABC Technologies | `hr@abctech.com` | Product Software (9.5 LPA) |
| **Company** | XYZ Corp | `hr@xyzcorp.com` | Hardware & Embedded (8.5 LPA) |
| **Company** | Apex Quant Solutions | `recruiter@apexquant.com` | Dream Tier Fintech (24.0 LPA) |
| **Coordinator** | Placement Cell | `coordinator@talenthub.edu` | Administrative Authority |

---

## 🧪 Recommended Evaluation Walkthrough

1. **Branch Eligibility Filtering**:
   - Sign in as **Sneha Patil** (`sneha@student.talenthub.edu` — Mechanical).
   - View the **Software Development Engineer** drive.
   - Click "View Criteria Breakdown" to inspect the live checklist: Mechanical is flagged as ineligible with explicit guidance.
2. **Multi-Round Interview Pipeline**:
   - Sign in as **Rahul Sharma** (`rahul@student.talenthub.edu`) and apply to **Software Development Engineer**.
   - Sign in as **ABC Technologies** (`hr@abctech.com`), navigate to applicants, shortlist Rahul, and schedule **Round 1 (Online Assessment)**.
   - Enter scorecard feedback and record a `PASS`. Notice that Round 2 only unlocks after Round 1 passes.
3. **Offer Gatekeeper**:
   - Verify that the formal offer button remains locked until all evaluation rounds are completed and passed.
4. **One-Offer Policy & Automatic Withdrawal**:
   - When Rahul accepts the 9.5 LPA offer from ABC Technologies, his placement status transitions to `PLACED`.
   - Any other active standard applications are automatically marked `WITHDRAWN_PLACED`.
   - Rahul is now barred from lower-tier drives and can only apply for **Dream Tier** drives (such as Apex Quant at 24 LPA).
