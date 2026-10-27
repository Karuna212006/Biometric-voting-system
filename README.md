# 🗳️ Biometric Fingerprint Voting System

An intelligent, full-stack **Biometric Electronic Voting System** designed to eliminate voter impersonation, fraudulent ballots, and double voting. Built with a **React 19 & Vite** frontend, a **Python Flask** REST API, **OpenCV ORB** computer vision for biometric fingerprint feature matching, **Appwrite Cloud** for document and identity proof storage, and **SQLite** for tamper-resistant ballot accounting.

---

## 📌 Table of Contents

- [Overview & Problem Statement](#-overview--problem-statement)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Biometric Matching Engine](#-biometric-matching-engine)
- [Tech Stack](#-tech-stack)
- [Project Directory Structure](#-project-directory-structure)
- [Database & Storage Schemas](#-database--storage-schemas)
- [API Endpoints](#-api-endpoints)
- [Getting Started & Installation](#-getting-started--installation)
  - [Prerequisites](#prerequisites)
  - [Backend Setup (Flask API)](#1-backend-setup-flask-api)
  - [Frontend Setup (React + Vite)](#2-frontend-setup-react--vite)
- [Configuration & Credentials](#-configuration--credentials)
- [End-to-End User Workflows](#-end-to-end-user-workflows)
- [Security & Integrity Protections](#-security--integrity-protections)
- [Future Roadmap](#-future-roadmap)
- [License & Acknowledgments](#-license--acknowledgments)

---

## 📖 Overview & Problem Statement

Traditional paper ballot and basic electronic voting systems remain vulnerable to:
1. **Voter Impersonation & Identity Fraud**: Fraudulent voting using forged identification cards.
2. **Double Voting**: An individual voting multiple times across different polling booths or shifts.
3. **Delayed Tallies & Human Error**: Inaccurate or slow ballot counting.

The **Biometric Fingerprint Voting System** addresses these vulnerabilities by introducing a multi-tiered verification pipeline:
- **Two-Factor Identity Check**: Requires the voter's unique voter ID number (verified against cloud-stored voter registration records with photo proof) combined with physical biometric fingerprint matching.
- **Computer Vision Verification**: Uses OpenCV's **ORB (Oriented FAST and Rotated BRIEF)** algorithm with Brute-Force Matcher to verify ridge and minutiae patterns against enrolled fingerprint templates.
- **One-Person, One-Vote Enforcer**: Enforces strict backend atomic state checking (`has_voted`), instantly disallowing subsequent vote attempts.

---

## ✨ Key Features

### 👤 Voter Experience
- **Role-Based Gateway**: Seamless entry portal for voters and election officials.
- **Identity Lookup**: Search voter registration records in real-time by Voter ID (`userId`).
- **Visual Photo Verification**: Displays the voter's official ID card image retrieved securely from Appwrite Cloud Storage for election officer cross-verification.
- **Biometric Ballot Casting**: Simple candidate selection coupled with biometric fingerprint upload for instant authentication and vote recording.
- **Tamper-Evident Status**: Instant feedback preventing already-voted citizens from re-voting.

### 🛡️ Administrative Portal
- **Protected Access**: Secure administrative authentication gate (`admin@gmail.com` / `admin123`) with session state management.
- **Real-Time Analytics Dashboard**:
  - Total registered voters counter.
  - Same-day registration tracker.
  - Demographic breakdown (Young voters aged 18–25 vs. voters aged 26+).
  - Live voter list table with automatic 10-second polling and visual countdown ticker.
- **Voter Registration Engine**:
  - Bulk multi-voter registration interface with dynamic form generation.
  - **Automated Age Verification**: Strict check calculating exact age from Date of Birth (rejects applicants under 18 years).
  - Dual enrollment: Uploads ID proof cards to **Appwrite Storage** and demographic records to **Appwrite Database**, while enrolling biometric templates into the **Flask SQLite backend**.
- **PDF Report Generation**: Instant export of registered voter rolls into formatted PDF documents using `jspdf` and `jspdf-autotable`.
- **Live Election Results**: Real-time progress bars, percentage calculations, candidate photos/party indicators, and live turnout monitoring.

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    subgraph Client["Frontend (React + Vite + Tailwind CSS)"]
        Hero["Gateway / Hero Screen (/)"]
        VoterUI["Voter Portal (/vote)"]
        AdminUI["Admin Dashboard (/admin-dashboard)"]
        RegisterUI["Voter Registration (/register-voters)"]
        ResultsUI["Live Results (/results)"]
    end

    subgraph Cloud["Appwrite Cloud Services"]
        AppwriteDB[("Appwrite Database\n- Voter Metadata\n(Name, DOB, Age, Address)")]
        AppwriteStorage[("Appwrite Storage Bucket\n- Voter ID Cards / Proofs")]
    end

    subgraph Backend["Python Flask Backend (:5000)"]
        FlaskRouter["Flask REST API (app.py)"]
        CVMatcher["OpenCV ORB Biometric Matcher"]
        PerlinGen["Perlin Noise Biometric Generator"]
        SQLiteDB[("SQLite Database\n(fingerprint_voting.db)\n- voters table\n- votes table")]
        ImgDir[("Local Fingerprints Storage\n(fingerprints/)")]
    end

    Hero -->|Voter Role| VoterUI
    Hero -->|Admin Role| AdminUI

    VoterUI -->|1. Lookup Voter ID| AppwriteDB
    AppwriteStorage -->|2. Fetch ID Image Proof| VoterUI
    VoterUI -->|3. Submit Biometric Ballot (POST /api/vote)| FlaskRouter

    RegisterUI -->|Save Demographic Data| AppwriteDB
    RegisterUI -->|Upload ID Photo| AppwriteStorage
    RegisterUI -->|Enroll Biometrics (POST /api/register)| FlaskRouter

    AdminUI -->|Fetch Voter Records| AppwriteDB
    ResultsUI -->|Fetch Vote Counts (GET /api/results)| FlaskRouter

    FlaskRouter --> CVMatcher
    CVMatcher --> ImgDir
    FlaskRouter --> SQLiteDB
```

---

## 🔬 Biometric Matching Engine

The core biometric verification runs inside [`backend/app.py`](file:///Fingerprint-voting-main/Fingerprint-voting-main/backend/app.py):

1. **Feature Detection & Extraction (ORB)**:
   - Uses `cv2.ORB_create()` to detect keypoints and compute rotational-invariant binary descriptors of the ridge characteristics in grayscale.
2. **Matching & Comparison (BFMatcher)**:
   - Uses `cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)` to calculate Hamming distances between query fingerprint descriptors and enrolled biometric templates.
3. **Threshold Scoring**:
   - Compares the candidate against all enrolled fingerprint files in the database.
   - Authentication succeeds only if the maximum number of matched keypoints exceeds the confidence threshold:
     $$\text{Matches} > 15$$
4. **Synthetic Testing Support**:
   - Contains a Perlin Noise synthetic pattern generator (`generate_fingerprint`) using 6 octaves and histogram equalization (`cv2.equalizeHist`) for mock fingerprint synthesis.

---

## 💻 Tech Stack

| Domain | Technology | Details / Version |
| :--- | :--- | :--- |
| **Frontend Framework** | React 19 | Fast UI components with modern hooks |
| **Build Tool** | Vite 6 | Lightning-fast HMR and bundling |
| **Styling** | Tailwind CSS 3.4 | Clean utility-first responsive layout |
| **Icons & Design** | React Icons (`react-icons`) | Feather, Heroicons, and Tabler icons |
| **Client Routing** | React Router DOM v7 | Single-page application route management |
| **HTTP Client** | Axios | REST communication with backend services |
| **Report Generation** | jsPDF & jspdf-autotable | Client-side voter roster PDF export |
| **Backend Framework** | Python 3 + Flask | Lightweight, robust REST service |
| **CORS Middleware** | Flask-CORS | Cross-origin resource sharing |
| **Computer Vision** | OpenCV (`opencv-python`) & NumPy | Image preprocessing, ORB extraction & matching |
| **Synthetic Biometrics**| Perlin Noise (`perlin-noise`) | Procedural ridge pattern synthesis |
| **Relational Database** | SQLite3 | Local storage of enrolled voters & vote counts |
| **Cloud BaaS** | Appwrite Cloud | Cloud storage for voter IDs & citizen profiles |

---

## 📁 Project Directory Structure

```text
Biometric-voting-system/
├── README.md                                # Project documentation (this file)
├── fingerprint_voting.db                     # SQLite database (votes & voter status)
├── fingerprints/                             # Enrolled fingerprint images directory
└── Fingerprint-voting-main/
    └── Fingerprint-voting-main/
        ├── backend/
        │   ├── app.py                       # Flask server, ORB matching, API routes
        │   ├── requirements.txt             # Python dependencies
        │   ├── fingerprint_voting.db        # Backend SQLite database
        │   ├── config/
        │   │   └── config.ini               # Database connection settings
        │   ├── fingerprints/                # Storage for enrolled fingerprint PNGs
        │   └── pics/                        # Sample test fingerprint images (finger1-9.jpg)
        └── frontend/
            ├── index.html                   # HTML entrypoint
            ├── package.json                 # Node dependencies and scripts
            ├── vite.config.js               # Vite build configuration
            ├── tailwind.config.js           # Tailwind CSS theme settings
            ├── appwriteConfig.js            # Appwrite SDK client & endpoint configuration
            ├── public/
            │   └── fingerprint.jpeg         # Static UI assets
            └── src/
                ├── App.jsx                  # Main application routing & auth guard
                ├── main.jsx                 # React root mount
                ├── index.css                # Global styles and Tailwind directives
                ├── assets/                  # Images and illustrations
                │   ├── admin.png
                │   └── manager.png
                ├── components/
                │   └── navbar.jsx           # Responsive dynamic navigation bar
                └── pages/
                    ├── hero.jsx             # Role selector landing page (Voter / Admin)
                    ├── home.jsx             # Voter home & election status board
                    ├── login.jsx            # Admin login screen
                    ├── adminDashboard.jsx   # Admin statistics, voter table & PDF export
                    ├── register.jsx         # Multi-voter registration form
                    ├── vote.jsx             # 2-step voter verification & ballot cast
                    └── result.jsx           # Live election results & tally progress
```

---

## 🗄️ Database & Storage Schemas

### 1. SQLite Database (`fingerprint_voting.db`)

#### Table: `voters`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | `TEXT` | `PRIMARY KEY` | Unique voter name / identifier |
| `fingerprint_path` | `TEXT` | `NOT NULL` | File path to stored fingerprint image |
| `has_voted` | `INTEGER` | `DEFAULT 0` | Flag: `0` = Not voted, `1` = Vote cast |

#### Table: `votes`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `candidate` | `TEXT` | `PRIMARY KEY` | Candidate or party name |
| `vote_count` | `INTEGER` | `DEFAULT 0` | Accumulated tally of votes |

*Default Candidates seeded on initialization:*
- `Tamilaga Vettri Kazhagam` (Candidate: Thalapathy Vijay)
- `DMK` (Candidate: Stalin)
- `NTK` (Candidate: Seeman)
- `BJP` (Candidate: Annamalai)

---

### 2. Appwrite Cloud Configuration
- **Endpoint**: `https://cloud.appwrite.io/v1`
- **Project ID**: `67fa7b4d00354918f84b`
- **Database ID**: `68107e44000a5ab2cc85`
- **Collection ID**: `681209580029497a3c02`
- **Storage Bucket ID**: `68107e5f001578ed6cc2`

#### Appwrite Voter Document Attributes:
- `userId` *(String)*: Unique Voter ID / EPIC number.
- `name` *(String)*: Full name of the citizen.
- `dob` *(String)*: Date of Birth (`DD-MM-YYYY`).
- `age` *(Integer)*: Calculated age (must be $\ge 18$).
- `address` *(String)*: Residential address.
- `voteridFileID` *(String)*: Appwrite Storage File ID for the uploaded ID card scan.

---

## 🔌 API Endpoints

The Flask server listens by default at `http://localhost:5000`:

| Method | Endpoint | Description | Request Payload | Response |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/register` | Enrolls one or multiple voters with fingerprints | `multipart/form-data`<br>• `voter_count` (int)<br>• `name_{i}` (str)<br>• `fingerprint_{i}` (file) | `{ "message": "Successfully registered N voters", "success": true }` |
| `POST` | `/api/vote` | Authenticates fingerprint and casts ballot | `multipart/form-data`<br>• `name` (str)<br>• `fingerprint` (file)<br>• `candidate` (str) | `{ "message": "Vote for <candidate> recorded successfully", "success": true }` |
| `GET` | `/api/results` | Retrieves current candidate vote counts | *None* | `{ "results": [{"candidate": "DMK", "vote_count": 5}, ...], "success": true }` |
| `GET` | `/api/voters` | Lists registered voters and voting status | *None* | `{ "voters": [{"name": "John", "has_voted": true}, ...], "success": true }` |

---

## 🚀 Getting Started & Installation

### Prerequisites

Ensure you have installed:
- [Node.js](https://nodejs.org/) (v18 or higher) and `npm`
- [Python](https://www.python.org/) (v3.9 or higher) and `pip`
- [Git](https://git-scm.com/)

---

### 1. Backend Setup (Flask API)

1. Open a terminal and navigate to the backend directory:
   ```bash
   cd Fingerprint-voting-main/Fingerprint-voting-main/backend
   ```

2. (Recommended) Create and activate a virtual environment:
   - **Windows**:
     ```powershell
     python -m venv venv
     .\venv\Scripts\activate
     ```
   - **Linux / macOS**:
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```

3. Install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

4. Start the Flask server:
   ```bash
   python app.py
   ```
   *The server will start at `http://localhost:5000` and automatically initialize `fingerprint_voting.db`.*

---

### 2. Frontend Setup (React + Vite)

1. Open a new terminal and navigate to the frontend directory:
   ```bash
   cd Fingerprint-voting-main/Fingerprint-voting-main/frontend
   ```

2. Install the Node modules:
   ```bash
   npm install
   ```

3. Start the Vite development server:
   ```bash
   npm run dev
   ```

4. Open your browser and visit:
   ```
   http://localhost:5173
   ```

---

## 🔑 Configuration & Credentials

### Default Administrative Credentials
| Role | Portal URL | Default Email | Default Password |
| :--- | :--- | :--- | :--- |
| **System Administrator** | `/login` | `admin@gmail.com` | `admin123` |

> [!NOTE]
> Admin credentials can be customized in [`frontend/src/pages/login.jsx`](file:///Fingerprint-voting-main/Fingerprint-voting-main/frontend/src/pages/login.jsx#L5-L6).

### Test Fingerprint Data
Sample test fingerprint images are provided in:
```
Fingerprint-voting-main/Fingerprint-voting-main/backend/pics/
  ├── finger1.jpg
  ├── finger2.jpg
  ├── finger3.jpg
  ├── ...
  └── finger9.jpg
```
Use these sample images when registering voters or casting test ballots.

---

## 🔄 End-to-End User Workflows

### 1. Admin Voter Enrollment Flow
1. Log in to the Admin Portal at `/login`.
2. Navigate to **Register Voters** (`/register-voters`).
3. Enter the number of voters to register.
4. Fill in:
   - Full Name
   - Date of Birth (System automatically validates $\text{Age} \ge 18$)
   - Voter ID Number
   - Residential Address
   - Voter ID Card Image Proof
   - Biometric Fingerprint Image
5. Click **Submit Registration**.
   - Profile & ID Card uploaded to **Appwrite**.
   - Biometric template registered with the **Flask SQLite backend**.

### 2. Voter Casting Ballot Flow
1. Select **Voter** from the Welcome screen (`/`).
2. Go to **Vote** (`/vote`).
3. **Step 1: ID Verification**:
   - Enter your Voter ID.
   - The system retrieves citizen details and displays the uploaded ID Card image for verification.
   - Click **Proceed to Vote**.
4. **Step 2: Biometric Ballot**:
   - Select the candidate of your choice.
   - Upload the voter's fingerprint scan.
   - Click **Cast My Vote**.
5. The backend executes ORB matching:
   - If match succeeds and `has_voted == 0`: Vote is tallied, voter is marked as voted.
   - If match fails: Verification error is displayed.
   - If already voted: Ballot is blocked.

### 3. Reviewing Results & Exporting Data
1. Log in as Admin and visit **Dashboard** (`/admin-dashboard`).
2. View real-time participation metrics and demographics.
3. Click **Export PDF** to generate an official voter roll PDF.
4. Visit **Results** (`/results`) to monitor live vote totals and percentage share per party.

---

## 🔒 Security & Integrity Protections

- **Anti-Impersonation**: Fingerprint descriptors are matched using ORB keypoint correspondence. A voter cannot vote without matching the physical biometric template stored during registration.
- **Double-Voting Prevention**: Once a vote is successfully cast, `has_voted` is set to `1` in SQLite. Any subsequent attempt immediately returns an error.
- **Age Validation**: Prevents under-age registration by verifying DOB against system calendar date.
- **Session Protection**: Administrative routes are guarded via `<ProtectedRoute>` preventing unauthorized access to voter records and election metrics.

---

## 🔮 Future Roadmap

- [ ] **Hardware Scanner Integration**: Direct integration with USB biometric devices (e.g., SecuGen, Mantra MFS100, Morpho) via WebUSB or native serial daemon.
- [ ] **Encrypted Storage**: AES-256 encryption for stored biometric templates.
- [ ] **Blockchain Audit Trail**: Immutable ledger integration for tamper-proof vote tally verification.
- [ ] **Multi-Booth Synchronization**: Distributed database cluster for multi-center election deployment.

---

## 📄 License & Acknowledgments

This project was developed for secure biometric-enabled electronic voting. 

- Developed by **Karuna** ([@Karuna212006](https://github.com/Karuna212006))
- Dependencies powered by OpenCV, Flask, React, and Appwrite.
