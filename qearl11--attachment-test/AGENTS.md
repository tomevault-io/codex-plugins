# Copilot Instructions

This file provides context and guidelines for GitHub Copilot to understand this project's specific conventions and architecture.

## Project Overview

**Project:** Personality Attachment Style Test (人格依恋类型测试)
**Goal:** A Chinese-language website to test and visualize attachment styles (Secure, Anxious, Avoidant, Disorganized) using a 2D chart.

## Tech Stack & Architecture

- **Languages:** JavaScript
- **Frameworks:** React
- **Styling:** CSS / Tailwind (suggested)
- **Visualization:** Recharts or Chart.js (for the 2D plot)

## Core Logic (Attachment Theory - ECR-RS)

The test appears to be based on the **ECR-RS (Experiences in Close Relationships-Relationship Structures)** questionnaire.

- **Structure:** 9 items per relationship domain (Mother, Father, Partner, General).
- **Dimensions:**
  - **Anxiety (焦虑):** Items related to fear of abandonment and insufficient love.
  - **Avoidance (回避):** Items related to discomfort with closeness and self-disclosure.
- **Scoring Rule:**
  - **Anxiety Items:** "I worry...", "I'm afraid...", "I often worry..."
  - **Avoidance Items:** "I prefer not to show...", "I don't feel comfortable..." (Normal scoring)
  - **Secure/Low Avoidance Items (Reverse Scored):** "I usually discuss...", "I talk things over...", "I find it easy to depend...", "It helps to turn to..."
  - **Calculation:**
    1. Reverse score the "Secure" items (e.g., if 1-7 scale: 8 - score).
    2. Calculate mean of Anxiety items.
    3. Calculate mean of Avoidance items.
    4. Plot on 2D chart (X: Avoidance, Y: Anxiety).

## Translation Guidelines

- **Tone:** Professional, psychological, empathetic (专业、心理学、共情).
- **Terminology:**
  - Secure -> 安全型
  - Anxious/Preoccupied -> 焦虑型
  - Avoidant/Dismissive -> 回避型
  - Disorganized/Fearful -> 恐惧型/混乱型
  - "Slide the button" -> "请滑动滑块" or "请选择"

## Data Structure

- Store questions in a structured format (JSON/JS) with properties: `id`, `text_en`, `text_zh`, `dimension` (anxiety/avoidance), `reverse` (boolean).

## Code Style & Conventions

- Use Functional Components and Hooks.
- **Localization:** All UI text must be in **Chinese**.
- **Comments:** Explain scoring algorithms clearly.

## Development Workflow

- **Build:** `npm run build`
- **Test:** `npm test`
- **Start:** `npm start`

## AI Interaction Guidelines

- When receiving English quiz content, translate it to natural, psychological Chinese context.
- Ensure the scoring logic accounts for reverse-scored items if present.
- Update this file as the project evolves to include specific patterns and architectural decisions.

---
> Source: [Qearl11/attachment-test](https://github.com/Qearl11/attachment-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-03 -->
