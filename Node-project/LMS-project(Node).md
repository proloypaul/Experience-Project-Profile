# 🎓 IT Training Tube - Advanced Learning Management System

![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Version](https://img.shields.io/badge/Version-1.0.0-blue?style=for-the-badge)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white)

Welcome to **IT Training Tube**, a professional, highly scalable, and full-featured Learning Management System (LMS) designed for modern online education and video-based training. This project was conceptualized, designed, and implemented independently by me around 2 years ago during my learning phase to showcase my full-stack web development capabilities.

---

## 🔗 Public Source Code & Live Links

This project is completely open-source. You can explore the codebase and live demonstrations below:

- **Backend GitHub Repository**: [\[Link to Backend Repo\] ](https://github.com/proloypaul/pnt-lms-server)

---

## 🚀 Key Features

### 1. 🖥️ Modern & Responsive User Interface
- **Pixel-Perfect Design**: Built with Tailwind CSS and Material Tailwind for a sleek, modern, and highly accessible user experience.
- **Responsive Layouts**: Fully responsive across mobile, tablet, and desktop devices ensuring seamless learning on the go.
- **Dynamic Interactions**: Smooth animations and transitions using Swiper carousels and React Hot Toast for instant user feedback.

### 2. 🔐 Advanced Authentication & Authorization
- **Next-Auth Integration**: Secure, robust session-based authentication on the frontend.
- **JWT Protection**: Backend API routes are highly protected using JSON Web Tokens (JWT) ensuring data integrity and security.
- **Role-Based Access Control (RBAC)**: Different access levels and dashboards for students, instructors, and administrators.

### 3. 🎥 High-Performance Video Streaming
- **Video Upload & Processing**: Seamless video uploads handled via Multer and transcoded using FFmpeg for optimal web delivery.
- **HLS Streaming**: Videos are streamed via HTTP Live Streaming (HLS) ensuring buffer-free playback across different network conditions.
- **Advanced Video Players**: Integrated with React Player and Vidstack for customizable, high-performance video playback.

### 4. 📚 Comprehensive Course Management
- **Rich Text Editing**: Instructors can create engaging, formatted course descriptions using the integrated React Quill rich text editor.
- **Interactive Quizzes & Assessments**: Features include countdown timers (React Countdown Circle Timer) to facilitate timed quizzes and skill assessments.
- **File Management**: Intuitive drag-and-drop file uploads utilizing Dropzone-UI for attaching course materials and resources.

### 5. 🛠️ Robust State Management & API Integration
- **Centralized State**: Leverages Redux Toolkit for predictable and efficient state management across complex frontend components.
- **Efficient Data Fetching**: Utilizes Axios for seamless, asynchronous communication between the Next.js frontend and the Express backend.

---

## 💻 Technology Stack

### Frontend Architecture 
- **Core Framework**: Next.js (v14)
- **UI Library**: React 18
- **Styling**: Tailwind CSS, Material Tailwind, Autoprefixer
- **State Management**: Redux Toolkit, React-Redux
- **Media & Streaming**: React Player, Vidstack, HLS.js
- **UI Components**: React Icons, React Select, SweetAlert2, React Spinners
- **Forms & Text**: React Quill (Rich Text Editor)
- **File Handling**: Dropzone-UI

### Backend Architecture 
- **Core Runtime**: Node.js
- **Framework**: Express.js
- **Language**: TypeScript (End-to-End Type Safety)
- **Database ORM**: Prisma (Compatible with PostgreSQL, MySQL, MongoDB)
- **Authentication**: JSON Web Token (JWT), Cookie Parser
- **Media Processing**: Fluent-FFmpeg, FFmpeg-Static, Multer
- **Code Quality**: ESLint, Prettier, Husky, Lint-Staged

---

## 📁 Folder Structure

This repository serves as a hub containing both the frontend and backend implementations in their respective directories:

```text
ittrainningtubeProject/
│
├── pnt-lms-template/                # Next.js Frontend Application
│   ├── public/                      # Static assets (images, fonts, etc.)
│   ├── src/                         # Source code (Components, Pages, Redux Slices, Utils)
│   ├── next.config.js               # Next.js framework configuration
│   ├── tailwind.config.js           # Tailwind CSS configuration
│   └── package.json                 # Frontend dependencies and scripts
│
├── ittrainingtube-server/
│   └── pnt-lms-server/              # Node.js + Express Backend API
│       ├── prisma/                  # Prisma schema and database migrations
│       ├── src/                     # Source code (Routes, Controllers, Middleware, Services)
│       ├── uploads/                 # Local directory for raw file uploads
│       ├── videos/                  # Local directory for processed HLS video files
│       ├── tsconfig.json            # TypeScript configuration
│       └── package.json             # Backend dependencies and scripts
│
└── README.md                        # Project root documentation (This File)
```

---

## 👨‍💻 Author & Implementation

**Developed and Implemented By**: Utsha

This project was built from the ground up to demonstrate advanced proficiency in architecting, developing, and deploying modern full-stack educational platforms. Feel free to explore the code