# Security Learning Game - Dynatrace App

## Architecture Overview

This is an educational gaming platform built with the **Dynatrace App Toolkit** (`dt-app`), using React + TypeScript. The app teaches application security through interactive scenarios, questions, and challenges.

### Key Components

- **Game Engine**: React Context-based state management for scenarios, progress, and scoring
- **Strato Components**: Use `@dynatrace/strato-components` and `@dynatrace/strato-components-preview` for UI
- **Educational Content**: Security scenarios with code examples, vulnerabilities, and explanations
- **Progress Tracking**: User sessions, leaderboards, and achievement systems

## Project Structure

```
ui/
├── main.tsx              # App entry point with AppRoot wrapper
├── app/
│   ├── App.tsx           # Main router with GameProvider context
│   ├── context/
│   │   └── GameContext.tsx   # Game state management
│   ├── components/
│   │   ├── scenario/     # Scenario display components
│   │   ├── questions/    # Question engine components
│   │   └── leaderboard/  # Leaderboard and scoring
│   ├── pages/            # Route components (Game, Leaderboard, etc.)
│   ├── types/
│   │   └── game.ts       # TypeScript interfaces for game data
│   └── data/
│       └── sampleData.ts # Sample scenarios and content
```

## Game Architecture Patterns

### State Management
```tsx
const { state, actions } = useGame();
// Access current user, scenarios, progress, leaderboard
// Actions: startScenario, submitAnswer, completeScenario
```

### Scenario Structure
- **Story Content**: Rich narrative with code examples and vulnerability explanations
- **Questions**: Multiple choice, code review, drag-drop challenges
- **Scoring**: Points-based system with explanations and hints
- **Prerequisites**: Locked scenarios that require completion of others

### Component Patterns
```tsx
// Scenario display with interactive code blocks
<ScenarioStory story={scenario.story} />

// Question engine supporting multiple question types
<QuestionContainer question={question} onAnswer={handleAnswer} />

// Leaderboard with rankings and achievements
<Leaderboard entries={leaderboard} currentUserId={userId} />
```

## Development Patterns

### Gaming State Flow
1. User selects scenario from `/game` page
2. `GameContext` manages session state and progress
3. Story display → Questions → Scoring → Results
4. Progress persisted and leaderboard updated

### Question Types
- `multiple-choice`: Single correct answer selection
- `multiple-select`: Multiple correct answers
- `code-completion`: Fill-in-the-blank coding
- `drag-drop`: Interactive element ordering
- `text-input`: Free-form answers
- `code-review`: Security vulnerability identification

### Security Education Focus
- Real-world vulnerability scenarios (SQL injection, XSS, etc.)
- Interactive code examples with syntax highlighting
- Business impact explanations and remediation guidance
- Progressive difficulty with prerequisite enforcement

## Development Workflow

### Commands
- `npm run start` - Dev server with hot reload
- `npm run build` - Production build
- `npm run deploy` - Deploy to Dynatrace environment

### Version Management (CRITICAL)
**ALWAYS increment version before deploying to avoid conflicts:**

1. **package.json** (line 3): Update `"version": "X.Y.Z"`
2. **app.config.json** (line 5): Update `"version": "X.Y.Z"`

**Example workflow:**
```bash
# Before deploying, increment version in both files:
# package.json: "0.11.0" → "0.12.0"
# app.config.json: "0.11.0" → "0.12.0"
npm run build
npm run deploy
```

**Failure to update versions causes:**
> `Cannot install app with version X.Y.Z because the same version is already installed with a different checksum.`

### Adding Content
1. Define scenarios in `data/sampleData.ts`
2. Create questions with multiple choice options
3. Include code examples and explanations
4. Set difficulty levels and prerequisites

### Data Integration
- Game state can integrate with Dynatrace storage APIs
- User progress and scoring persistence
- Real-time leaderboard updates via DQL queries

### Leaderboard Implementation
- **Ranking Logic**: Based on first attempt scores (fairest competition)
- **Display Format**: Shows first attempt score with best score in brackets when different
- **Real-time Updates**: DQL queries with 30-second refresh cycle
- **Results Preview**: Shows predicted leaderboard position immediately after game completion
- **Data Source**: Business events from Grail with proper deduplication and rate limiting

## Educational Content Structure

### Scenario Categories
- **injection**: SQL injection, NoSQL injection
- **xss**: Cross-site scripting variants
- **authentication**: Auth bypass, session management
- **encryption**: Cryptographic failures
- **csrf**: Cross-site request forgery
- **components**: Vulnerable dependencies

### Learning Progression
- Beginner → Intermediate → Advanced → Expert
- Prerequisites ensure proper learning sequence
- Achievement system rewards mastery and consistency



### DQL Best Practices

Follow [official DQL Best Practices](https://docs.dynatrace.com/docs/discover-dynatrace/platform/grail/dynatrace-query-language/dql-best-practices) for optimal query performance:

#### Core Rules:
- **Always specify time range**: `fetch bizevents, from:now() - 30d`
- **Filter early**: Apply filters immediately after fetch for best performance
- **Use proper string matching**: `==` for exact match, `matchesPhrase()` for substring search
- **Correct summarize syntax**: `summarize count = count(), by:{field}` (note the curly braces)
- **Follow command order**: filter → fields → summarize → sort → limit

#### String Matching:
- Use `==` or `!=` when field value is known exactly
- Use `matchesPhrase(field, "text")` for substring search (NOT `contains()`)
- Use `startsWith(field, "prefix")` and `endsWith(field, "suffix")` for prefix/suffix matching

#### Performance Optimization:
- **Narrow time ranges**: Shorter windows = better performance
- **Filter on indexed fields**: `event.type`, `event.provider`, `dt.system.bucket`
- **Use sampling for logs**: `fetch logs, samplingRatio:100` for large datasets
- **Limit scanned data**: `scanLimitGBytes:100` parameter when needed

### DQL Usage Guide

Based on the [official DQL Guide](https://docs.dynatrace.com/docs/discover-dynatrace/platform/grail/dynatrace-query-language/dql-guide):

#### Pipeline Structure:
- **Commands are chained with pipe operator** (`|`)
- **Data flows sequentially** from one command to the next
- **Order matters** for both performance and results

#### Basic Query Pattern:
```
fetch [data_type], from:now() - [timeframe]
| filter [conditions]
| fields [field_list] 
| summarize [aggregations], by:{[grouping_fields]}
| sort [field] [asc|desc]
| limit [number]
```

#### Time Specification Examples:
- Relative: `from:now() - 2h`, `from:now() - 24h, to:now() - 2h`
- Absolute: `timeframe:"2021-10-20T00:00:00Z/2021-10-28T12:00:00Z"`

## Business Events Integration Status

### Current Implementation vs Best Practices
Based on [official Dynatrace documentation](https://developer.dynatrace.com/develop/data/ingest-data/#business-events):

**✅ FULLY COMPLIANT:**
- ✅ Uses official `businessEventsClient` from `@dynatrace-sdk/client-classic-environment-v2`
- ✅ Correct content-type: `application/json; charset=utf-8`
- ✅ Proper event structure with event.type and timestamp
- ✅ SDK handles authentication automatically
- ✅ SDK handles retries and error handling
- ✅ Rate limiting and deduplication logic
- ✅ Input validation and error handling

**📋 Implementation Details:**
```typescript
import { businessEventsClient } from '@dynatrace-sdk/client-classic-environment-v2';

export async function sendGameEvent(eventData: GameCompletionEvent): Promise<void> {
  // Event deduplication and rate limiting
  // Input validation
  
  const businessEvent = {
    "event.type": "com.dynatrace.security.game.session.completed",
    "event.provider": "TotalServiceMeltdown",
    timestamp: new Date().toISOString(),
    ...eventData
  };

  // Official SDK usage - handles auth, retries, proper headers
  await businessEventsClient.ingest({
    body: businessEvent,
    type: 'application/json; charset=utf-8'
  });
}
```

## Business Events Query Patterns
```
// Leaderboard aggregation
fetch bizevents, from:now() - 30d
| filter event.type == "com.dynatrace.security.game.session.completed"
| fields email, firstname, score, timestamp
| summarize attempts = count(), maxScore = max(score), by:{email}
| sort maxScore desc
| limit 50
```

## Log Analysis Patterns
```
fetch logs, from:now() - 2h
| filter loglevel == "ERROR"
| fields timestamp, content, loglevel, k8s.pod.name, exception.message
| sort timestamp desc
| limit 20
```

## Problem Investigation Patterns
```
fetch events, from:now() - 24h
| filter event.kind == "DAVIS_PROBLEM"
| fields timestamp, display_id, event.name, event.status, event.description
| sort timestamp desc
| limit 20
```

## Security Events Analysis
```
fetch security.events, from:now() - 7d
| filter dt.system.bucket == "default_securityevents_builtin"
  and event.provider == "Dynatrace"
  and event.type == "VULNERABILITY_STATE_REPORT_EVENT"
| summarize count = count(), by:{vulnerability.risk.level}
| sort count desc
```

## Field Discovery
```
fetch dt.semantic_dictionary.fields
| filter matchesPhrase(name, "k8s")
| fields name, description, examples
| limit 20
```


## Dynatrace Business Events Integration

### Overview
The game integrates with Dynatrace Grail to track game completions and display real-time leaderboards using business events.

### Business Event Schema
```typescript
interface GameCompletionEvent {
  "event.type": "com.dynatrace.security.game.session.completed";
  "event.provider": "TotalServiceMeltdown";
  timestamp: string; // ISO 8601 format
  firstname: string;
  email: string;
  score: number;      // Points earned (0-10000)
  seconds: number;    // Time taken to complete
  attempts: number;   // Which attempt this was for the user
}
```

### Sending Events
```typescript
import { sendGameEvent } from '../services/grailService';

// Send game completion event
await sendGameEvent({
  firstname: "John",
  email: "john@example.com", 
  score: 8500,
  seconds: 180,
  attempts: 1
});
```

### Reading Events with DQL
```typescript
import { useDql } from "@dynatrace-sdk/react-hooks";

const DQL = `
  fetch bizevents, from:now() - 30d
  | filter event.type == "com.dynatrace.security.game.session.completed"
  | fields email, firstname, score, timestamp
  | summarize attempts = count(), maxScore = max(score), firstAttemptScore = min(score), firstname = min(firstname), by:{email}
  | fields firstname, maxScore, attempts, firstAttemptScore
  | sort maxScore desc 
  | limit 50
`;

const { data, error, isLoading } = useDql({ 
  query: DQL,
  refetchInterval: 30000 
});
```

### Error Handling
- **Event deduplication**: Prevents duplicate submissions using localStorage
- **Rate limiting**: 5-second cooldown between events
- **Input validation**: Validates firstname and email format
- **Graceful degradation**: Shows user-friendly error messages

### Best Practices
1. **Always validate user inputs** before sending events
2. **Use unique keys** for event deduplication
3. **Handle network errors** gracefully with user feedback
4. **Rate limit** to prevent spam submissions
5. **Test with real data** to verify Grail integration works

---
> Source: [nikhilgoenkatech/adoptionday](https://github.com/nikhilgoenkatech/adoptionday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-09 -->
