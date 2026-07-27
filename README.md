# Gesturix: Multimodal Communication for the Deaf & Mute Community

A real-time, AI-powered platform that bridges communication between deaf/mute and hearing participants during online meetings — translating sign language to text and speech to text, simultaneously, in one web app.

> **Note:** This repository documents the project and architecture. The full implementation is currently kept private; a working demo/walkthrough is available on request.

## Problem Statement
Deaf and speech-impaired individuals face significant communication barriers in education, healthcare, and daily life due to the lack of a universal, real-time communication medium. Existing solutions are typically:
- Unidirectional (translate in only one direction)
- Not real-time or integrated with common video-conferencing tools
- Lacking multimodal interaction and multilingual support

**Gesturix** is an AI-powered, real-time, bidirectional translation bridge between Sign Language, Speech, and Text — built entirely on open-source AI models and standard webcams, with no specialized hardware required.

## Why This Matters
- **Accessible real-time communication:** lets deaf and mute individuals participate in video meetings through live sign-to-text translation and speech-to-text captioning, reducing dependence on human interpreters
- **Cost-effective:** built on open-source AI (MediaPipe Holistic, a custom Transformer model) plus standard webcams — no depth cameras or specialized hardware
- **Digital inclusion:** supports equal participation in education, healthcare, and professional settings, aligned with UN Sustainable Development Goal 10 (Reduced Inequalities)

## Core Objectives
- **Accurate sign recognition** — robust ASL gesture interpretation across 250 vocabulary classes
- **Real-time speech transcription** — live English captioning
- **Platform integration** — seamless operation inside WebRTC video conferencing (Daily.co)
- **Optimized performance** — real-time system achieving 200–400ms sign-to-text latency and <500ms WebSocket response time, with scalable multi-stream processing
- **Unified UX** — a single accessible interface for both deaf and hearing participants, with color-coded dual-modality captions

## System Architecture — Three-Layer Design

### Layer 1 — Real-Time Integration Layer
- Captures live video (30 FPS) and audio from webcam & microphone
- **MediaPipe Holistic** extracts 75 body + hand landmarks in real time
- A 7-stage feature engineering pipeline converts landmarks into a **1122-dimensional feature vector** (temporal interpolation, pose normalization, hand-relative coordinates, velocity + acceleration)
- Routes processed data to the AI layer and manages real-time WebSocket output

### Layer 2 — AI Processing Layer
**Sign-to-Text:**
- **GesturixPro Transformer** — a custom 5-block Pre-LN Transformer with multi-head attention pooling
- 250 ASL word classes, **92.24% known-class accuracy**
- 5–15ms inference speed on GPU
- **Three-tier grammar correction:** instant pattern matching (0ms) → rule-based rewriting (~1ms) → Qwen2.5-0.5B-Instruct LLM refinement (1–2s, runs in background)

**Speech-to-Text:**
- Azure Cognitive Services Speech SDK in continuous recognition mode
- Native webm/opus support via GStreamer, 1.5–3 second latency

**Real-Time Broadcast:**
- 3-channel WebSocket architecture: Caption Hub (speech), Sign Detection (sign), and Chat
- In-memory, room-based client management; <500ms broadcast latency

### Layer 3 — Backend & Infrastructure Layer
- **Backend:** FastAPI + Google OAuth 2.0 + Daily.co WebRTC
- **Database:** Supabase PostgreSQL, indexed for cross-device sync
- **Deployment:** React 18 frontend on Vercel; FastAPI backend on Modal.com GPU cloud (NVIDIA T4)
- **Scale:** 5 concurrent meeting containers, 100 WebSocket connections per container

## Tech Stack
- **AI & Core ML:** PyTorch, MediaPipe Holistic, NumPy, OpenCV, Qwen2.5-0.5B-Instruct, Azure Speech SDK, FastAPI & WebSockets
- **Languages & Data:** Python, NumPy, Pandas, OpenCV
- **Development & Training:** Jupyter/Google Colab, Git
- **Backend & APIs:** FastAPI, Azure Cognitive Services (Speech SDK), Daily.co
- **Frontend:** React + TypeScript, deployed on Vercel

## Dataset & Related Work
Built on the **Google ISLR dataset** (94,477 recordings, 250 classes, Parquet landmarks) — chosen for being more compact and GPU-efficient than larger benchmarks like WLASL (21,000+ clips, 2,000 classes) or MS-ASL. The pose-estimation approach follows MediaPipe Holistic (Bazarevsky et al., 2020; Lugaresi et al., 2019), and the sequence model builds on Transformer self-attention (Vaswani et al., 2017), addressing the real-time latency limitations found in prior sign-language Transformers (Camgöz et al., 2020). Unlike existing commercial tools (Google Meet/Teams captions with no sign-language input, or SignAll's hardware-dependent ASL translation), Gesturix is the first platform to combine sign-to-text, speech-to-text, and WebRTC video conferencing in a single web app.

## Primary User Journey
1. **Deaf user (signs):** Webcam → MediaPipe Holistic (75 landmarks) → Feature engineering (1122-D vector) → GesturixPro Transformer → Confidence & entropy filtering → Grammar correction (rules + Qwen2.5-0.5B-Instruct) → Live caption display → WebSocket broadcast
2. **Hearing user (speaks):** Microphone → Azure Speech SDK → Speech-to-text → Live caption display → WebSocket broadcast
3. **Real-time integration:** Both caption streams flow through the FastAPI backend + WebSocket manager into the Daily.co meeting interface for live multimodal communication
4. **Data persistence:** Captions are stored in Supabase PostgreSQL for transcript storage and retrieval

## Key Features
- Secure sign-in via Google OAuth
- Create/join meetings through Daily.co integration
- Real-time ASL sign captioning for hearing participants
- Real-time speech-to-text captioning for deaf/mute participants
- In-meeting chat messaging
- Meeting transcripts and captions exportable as PDF
- Cross-device transcript storage and retrieval via Supabase

## Applications
- **Healthcare:** doctor-patient consultations, telemedicine, pharmacy instructions
- **Workplace:** job interviews, meetings & training, customer service
- **Education:** inclusive classroom lectures, parent-teacher meetings, study group support
- **Government services:** police stations & legal proceedings, public service counters, voting assistance
- **Digital & social:** video calls, social media live streams, gaming communication

## Challenges Addressed
- Real-time sign recognition from live webcam streams
- Handling variation in signing speed, hand positions, and gesture styles
- Maintaining low latency through WebSockets and cloud GPU deployment
- Synchronizing speech captions, sign captions, and chat in a single meeting environment
- Cross-device transcript storage and retrieval

## Roadmap
**Short-term (6–12 months):** expand ASL vocabulary beyond 250 classes, improve accuracy/robustness in real-world conditions, optimize inference latency, enhance transcript management (search, filtering, summaries)

**Mid-term (1–3 years):** support additional sign languages, multilingual speech/caption translation, scale to larger concurrent meetings, dedicated mobile apps (Android/iOS)

**Long-term (3–5 years):** real-time cross-language communication between signers and speakers, AI-powered contextual translation, integration with education/healthcare/workplace accessibility platforms, global deployment

## Team
Final Year Project, NED University of Engineering & Technology

**Supervisor:** Dr. Usman Amjad
**Co-Supervisor:** Miss Mehar Fatima
