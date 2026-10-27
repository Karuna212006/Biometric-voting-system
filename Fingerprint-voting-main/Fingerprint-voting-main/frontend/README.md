# 🗳️ Fingerprint Voting System - Frontend

This is the React + Vite frontend application for the **Biometric Fingerprint Voting System**.

For complete system documentation, architecture diagrams, backend setup, and biometric algorithms, refer to the [Root README.md](../../../README.md).

---

## ⚡ Tech Stack

- **Framework**: [React 19](https://react.dev/)
- **Bundler**: [Vite 6](https://vitejs.dev/)
- **Styling**: [Tailwind CSS 3.4](https://tailwindcss.com/)
- **Icons**: [React Icons](https://react-icons.github.io/react-icons/)
- **Routing**: [React Router DOM v7](https://reactrouter.com/)
- **Cloud Backend**: [Appwrite Web SDK 17](https://appwrite.io/)
- **PDF Generation**: [jsPDF](https://github.com/parallax/jsPDF) & [jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable)
- **HTTP Client**: [Axios](https://axios-http.com/)

---

## 🚀 Quick Start

### 1. Install Dependencies
```bash
npm install
```

### 2. Run the Development Server
```bash
npm run dev
```

The frontend will run at:
```
http://localhost:5173
```

Ensure the Flask backend is also running on port `5000` (`http://localhost:5000`).

---

## 🔐 Admin Credentials

- **Email**: `admin@gmail.com`
- **Password**: `admin123`
- **Login Route**: `/login`

---

## 📂 Routes Overview

| Route | Access | Component | Description |
| :--- | :--- | :--- | :--- |
| `/` | Public | `<Hero />` | Gateway screen to select Voter or Admin role |
| `/home` | Public | `<HomePage />` | Voter home & election status board |
| `/vote` | Public | `<VotePage />` | 2-step voter verification and biometric ballot cast |
| `/login` | Public | `<LoginPage />` | Admin login page |
| `/admin-dashboard` | Protected | `<AdminDashboard />` | Registered voter analytics, polling table, and PDF export |
| `/register-voters` | Protected | `<RegisterPage />` | Bulk voter registration with age validation |
| `/results` | Protected | `<ResultsPage />` | Live election vote counts & percentage progress |
