# GitHub Copilot Instructions - RoboTorq Network

**Project**: RoboTorq - Physics-Based Monetary System  
**Last Updated**: November 15, 2025  
**Purpose**: Guide AI agents through the proven development workflow

---

## 🎯 Project Overview

RoboTorq is **NOT** a cryptocurrency—it's a NATS-based distributed system where robotic labor creates measurable value through physics.

**Core Equation**: `1 RoboTorq = 1 kWh × 3600 tokens/sec × 1 hour`

**Architecture**: Message-passing system (NATS pub/sub), not blockchain
- **No chain**: Real-time message flows, not append-only ledger
- **No mining**: Value minted based on verified work
- **No gas fees**: Transaction fees fund operations (demurrage)

**Critical Reading**:
1. `VAULT_IMPLEMENTATION_PLAN.md`: Overall SPRINT architecture
    - `src/vault/VAULT_ARCHITECTURE.md` also serves as doc template for all docs created while implementing the plan
2. `BRANCHING.md`: Git workflow (`v0` baseline, `feature/*` branches)
3. **`port mapping/PORT_MAPPINGS.md`**: **MANDATORY** - Check BEFORE any port changes
4. Service-specific architecture docs:
   - `src/mint/MINT_ARCHITECTURE.md`
   - `src/refinery/REFINERY_ARCHITECTURE.md`
   - `src/trust/TRUST_ARCHITECTURE.md` (future)

---

## 🔄 The Power Workflow

This 14-step cycle achieved **95% completion** of the currency refactor in one focused sprint.

### Phase 1: Planning & Context

#### 1. **Branch Checkout**
```bash
# Always work on feature branches
git checkout -b feature/currency-refactor

# Or continue existing work
git checkout feature/your-feature
git pull origin feature/your-feature
```

**Why**: Isolates work, enables parallel development, protects `main` from breaking changes.

#### 2. **Architecture Review**
Read relevant architecture docs BEFORE coding:
```bash
# For Mint work
cat src/mint/MINT_ARCHITECTURE.md

# For Refinery work
cat src/refinery/REFINERY_ARCHITECTURE.md

# For cross-service changes
grep -r "data flow" src/*/ARCHITECTURE.md
```

**Focus Areas**:
- Component responsibilities (what does X do?)
- Data flow diagrams (how do services communicate?)
- Key data structures (JouleTorqUnit, TokenTorqIngot, RoboTorqBatch)
- Configuration patterns (env vars, defaults)

**Example from Currency Refactor**:
> "Mint aggregates 1000 ingots OR flushes after 60s, builds merkle tree from ingot hashes, publishes batch to DistoDam."

#### 3. **Code Pattern Analysis**
Study existing implementations to match patterns:
```bash
# Example: Understanding Mint's components
find src/mint/internal/mint -name "*.go" | xargs head -50

# Look for:
# - Constructor patterns (New*())
# - Interface definitions
# - Error handling
# - Logging style
# - Metrics instrumentation
```

**Key Patterns in RoboTorq**:
- **Dependency Injection**: Components passed via constructors, not globals
  ```go
  func NewMintEngine(logger *slog.Logger, metrics *Metrics) *MintEngine
  ```

- **Graceful Shutdown**: All services respect `context.Context`
  ```go
  func (s *Service) Start(ctx context.Context) {
      for {
          select {
          case <-ctx.Done():
              s.drainBuffer()  // Flush pending work
              return
          // ...
          }
      }
  }
  ```

- **Structured Logging**: JSON logs with contextual fields
  ```go
  slog.Info("ingot assembled",
      "ingot_id", ingot.ID,
      "units", len(ingot.Units),
      "joules", ingot.JouleTorqTotal)
  ```

- **Prometheus Metrics**: Every operation instrumented
  ```go
  type Metrics struct {
      IngotsReceivedTotal prometheus.Counter
      ProcessingLatency   prometheus.Histogram
  }
  
  m.IngotsReceivedTotal.Inc()
  ```

---

### Phase 2: Implementation

#### 4. **Create TODOs in Code**
Mark implementation points BEFORE building:
```go
// TODO(currency-refactor): Replace dual queues with single unit queue
// - Remove: jouleQueue, roboQueue
// - Add: unitQueue []JouleTorqUnit
// - Update: processOre() to extract units
// - Fix: assembleIngot() to consume units
// - Test: Multi-contract unit aggregation

type Refinery struct {
    // OLD
    jouleQueue []float64  // TODO: REMOVE
    roboQueue  []float64  // TODO: REMOVE
    
    // NEW
    unitQueue  []*JouleTorqUnit  // TODO: IMPLEMENT
}
```

**Why**: Creates roadmap, enables incremental commits, documents intent.

**Naming Convention**:
- `TODO(feature-name):` - Planned work on feature branch
- `FIXME(issue-123):` - Bug fix for GitHub issue
- `WIP(component):` - Work in progress, incomplete
- `HACK:` - Temporary workaround, needs cleanup

#### 5. **Execute TODOs (Implementation)**
Tackle one TODO at a time, test as you go:

**Example: Refinery Single Queue Implementation**

```go
// Step 1: Add new data structure
type QueueManager struct {
    units    []*models.JouleTorqUnit
    mu       sync.Mutex
    notEmpty *sync.Cond
}

// Step 2: Implement AddUnit
func (qm *QueueManager) AddUnit(unit *models.JouleTorqUnit) error {
    qm.mu.Lock()
    defer qm.mu.Unlock()
    
    qm.units = append(qm.units, unit)
    qm.notEmpty.Signal()
    return nil
}

// Step 3: Implement GetUnit (blocking)
func (qm *QueueManager) GetUnit() (*models.JouleTorqUnit, error) {
    qm.mu.Lock()
    defer qm.mu.Unlock()
    
    for len(qm.units) == 0 {
        qm.notEmpty.Wait()  // Block until unit available
    }
    
    unit := qm.units[0]
    qm.units = qm.units[1:]
    return unit, nil
}
```

**Incremental Testing**:
```bash
# Test each function as implemented
go test ./internal/refinery/queue_manager_test.go -run TestAddUnit -v
go test ./internal/refinery/queue_manager_test.go -run TestGetUnit -v
go test ./internal/refinery/queue_manager_test.go -run TestConcurrency -v
```

#### 6. **Create Tests (TDD Approach)**
Write tests ALONGSIDE implementation, not after:

**Unit Test Example** (`queue_manager_test.go`):
```go
func TestQueueManager_AddAndGet(t *testing.T) {
    qm := NewQueueManager(100)
    
    unit := &models.JouleTorqUnit{
        TokenID: "test-token-001",
        JoulesConsumed: 4.17,
    }
    
    // Test add
    err := qm.AddUnit(unit)
    assert.NoError(t, err)
    assert.Equal(t, 1, qm.Len())
    
    // Test get
    retrieved, err := qm.GetUnit()
    assert.NoError(t, err)
    assert.Equal(t, "test-token-001", retrieved.TokenID)
    assert.Equal(t, 0, qm.Len())
}

func TestQueueManager_BlockingGet(t *testing.T) {
    qm := NewQueueManager(100)
    
    retrieved := false
    go func() {
        unit, _ := qm.GetUnit()  // Blocks
        assert.Equal(t, "async-token", unit.TokenID)
        retrieved = true
    }()
    
    time.Sleep(100 * time.Millisecond)
    assert.False(t, retrieved, "Should still be blocked")
    
    qm.AddUnit(&models.JouleTorqUnit{TokenID: "async-token"})
    
    time.Sleep(100 * time.Millisecond)
    assert.True(t, retrieved, "Should have unblocked")
}
```

**Coverage Target**: 95%+ for all new code
```bash
go test ./... -cover

# Expected output:
# ok      refinery/internal/refinery  0.450s  coverage: 96.2% of statements
```

---

### Phase 3: Validation

#### 7. **Step-by-Step Commits**
Commit after EACH working increment (not at end of day):

**Commit Pattern**:
```bash
# After implementing QueueManager AddUnit
git add internal/refinery/queue_manager.go
git add internal/refinery/queue_manager_test.go
git commit -m "feat(refinery): Add QueueManager.AddUnit with thread safety

- Implements unit storage in slice
- Uses mutex for goroutine safety
- Signals condition variable on add
- Test: concurrent adds from 10 goroutines"

# After implementing GetUnit
git add internal/refinery/queue_manager.go
git add internal/refinery/queue_manager_test.go
git commit -m "feat(refinery): Add QueueManager.GetUnit with blocking

- Blocks when queue empty (no busy-waiting)
- Uses sync.Cond for efficient wakeup
- Thread-safe pop operation
- Test: blocking behavior verified"

# After integration test
git add internal/refinery/integration_test.go
git commit -m "test(refinery): Add QueueManager integration test

- Tests full ore → unit → queue flow
- Validates concurrent AddUnit/GetUnit
- Confirms FIFO ordering
- Coverage: 100% of QueueManager"
```

**Commit Message Format** (Conventional Commits):
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `test`: Adding/updating tests
- `refactor`: Code restructure (no behavior change)
- `docs`: Documentation only
- `chore`: Build, CI, dependencies

**Scopes**: `mint`, `refinery`, `trust`, `distodam`, `digger`, `wallet`, `printer`

**Real Example from Currency Refactor**:
```
feat(refinery): Implement unit-based ingot assembly

Replace dual joule/robo queues with single JouleTorqUnit queue:
- Add QueueManager with AddUnit/GetUnit
- Update IngotAssembler to accumulate units
- Build merkle branch hash in NewTokenTorqIngot
- Preserve complete proof chain (token → unit → ingot)

Breaking Change: Ore processing now creates 1 unit per token
instead of aggregating joules/robo separately.

Tests:
- queue_manager_test.go: 100% coverage
- ingot_assembler_test.go: 96% coverage
- integration_test.go: Full pipeline verified

Refs: feature/currency-refactor, README.md Appendix O
```

#### 8. **Conduct E2E Tests**
Validate COMPLETE pipeline after changes:

**RoboTorq E2E Test** (`test-digger-e2e.ps1`):
```powershell
# Start all services
docker-compose up -d

# Wait for health
Start-Sleep -Seconds 10

# Build headless Digger
cd src/digger-app/digger
cargo build --release --bin headless

# Run headless in background
Start-Process -NoNewWindow -FilePath "./target/release/headless.exe"

# Send stake + execute contract
Invoke-RestMethod "http://localhost:9000/stake" -Method Post -Body '{"amount":0.05}' -ContentType "application/json"
Invoke-RestMethod "http://localhost:9000/execute" -Method Post -Body '{"contract_id":"e2e-test"}' -ContentType "application/json"

# Wait for processing (13 ores @ 5s each = 65s)
Write-Host "⏳ Waiting 65 seconds for ore processing..."
Start-Sleep -Seconds 65

# Verify Refinery logs
$logs = docker logs robotorq-network-refinery-1 --since 70s

if ($logs -match "ingot assembled") {
    Write-Host "✅ SUCCESS: Ingot assembled!" -ForegroundColor Green
    
    # Extract details
    $ingotLine = $logs | Select-String "ingot assembled" | Select-Object -Last 1
    Write-Host $ingotLine
} else {
    Write-Host "❌ FAILURE: No ingot found" -ForegroundColor Red
    exit 1
}

# Cleanup
Stop-Process -Name "headless" -Force
docker-compose down
```

**Expected Output**:
```
⏳ Waiting 65 seconds for ore processing...
✅ SUCCESS: Ingot assembled!

{"msg":"ingot assembled",
 "ingot_id":"ingot-20251115-210012.547794",
 "joules":14999.999999999249,
 "robo_stake":0.050000000000002334,
 "units":3600,
 "contracts":["e2e-test-contract-001"],
 "branch_hash":"0521434e13fa9bddc71d777d689635f73b1d91df68b74abcbd97af58a051ff10"}
```

**What E2E Test Validates**:
- ✅ Digger executes contract, sends ore
- ✅ Refinery receives ore, extracts units
- ✅ QueueManager stores units
- ✅ IngotAssembler accumulates 3600 units
- ✅ NewTokenTorqIngot builds merkle hash
- ✅ BatchSender publishes to Mint

#### 9. **Update Documentation**
Sync docs with implementation BEFORE merging:

**Architecture Doc Updates**:
```markdown
## Component Architecture

### QueueManager - Single Unit Queue ✅ UPDATED

**Responsibilities**:
- Store JouleTorqUnits in thread-safe FIFO queue
- Block on GetUnit() when empty (no busy-waiting)
- Track queue depth metrics

**Architecture Change** (Currency Refactor):
- ❌ OLD: Dual queues (JouleQueue + RoboQueue) - lost token granularity
- ✅ NEW: Single queue ([]JouleTorqUnit) - preserves complete proof chain

**Implementation**:
[...code examples...]

**Tests**: `queue_manager_test.go` (100% coverage)
```

**README Updates**:
```markdown
## Appendix O: Data Structures

### JouleTorqUnit ✅ UPDATED

The **atomic unit** of the RoboTorq system. Every token becomes one unit.

```go
type JouleTorqUnit struct {
    TokenID        string    `json:"token_id"`         // Unique: {contract}-m{milestone}-t{index}
    ContractID     string    `json:"contract_id"`      
    DiggerID       string    `json:"digger_id"`        
    JoulesConsumed float64   `json:"joules_consumed"`  // Energy for THIS token
    RoboStakePaid  float64   `json:"robo_stake_paid"`  // Cost of THIS token
    MilestoneIndex int       `json:"milestone_index"`  
    Timestamp      time.Time `json:"timestamp"`        
    Hash           string    `json:"hash"`             // SHA256 of all fields
    Signature      []byte    `json:"signature"`        // Dilithium5 signature
}
```

**Flow**: JouleTorqOre (300 tokens) → 300 JouleTorqUnits → accumulate to 3600 → TokenTorqIngot
```

**Commit Docs with Code**:
```bash
git add src/refinery/REFINERY_ARCHITECTURE.md
git add README.md
git commit -m "docs(refinery): Update architecture for unit-based queue

- Document QueueManager single-queue design
- Explain unit extraction from ore
- Add merkle hash construction details
- Update Appendix O with JouleTorqUnit flow"
```

---

### Phase 4: Integration

#### 10. **Update Build and CI**
Ensure CI can build and test your changes:

**Update Dockerfile** (if dependencies changed):
```dockerfile
FROM golang:1.24-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download  # ← Downloads new dependencies

COPY . .
RUN CGO_ENABLED=0 go build -o refinery ./cmd/refinery

# Tests run in CI
RUN go test ./... -cover

FROM alpine:3.20
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/refinery .
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:8081/health || exit 1
CMD ["./refinery"]
```

**Update `docker-compose.yaml`** (if new env vars):
```yaml
services:
  refinery:
    build: ./src/refinery
    environment:
      - NATS_URL=nats://nats:4222
      - REFINERY_JOULE_QUEUE_SIZE=1000        # ← NEW
      - REFINERY_INGOT_BATCH_INTERVAL=60      # ← NEW
      - LOG_LEVEL=info
    depends_on:
      - nats
```

**Verify Local Build**:
```bash
# Build image
cd src/refinery
docker build -t refinery:test .

# Run tests in container
docker run --rm refinery:test go test ./... -v

# Start service
docker-compose up -d refinery

# Check health
curl http://localhost:8081/health
```

#### 11. **Push to GitHub for Testing**
Trigger CI on feature branch:

```bash
# Ensure all tests pass locally first
go test ./... -v
pwsh test-digger-e2e.ps1

# Push feature branch
git push origin feature/currency-refactor
```

**GitHub Actions CI** (`.github/workflows/ci.yml`):
```yaml
name: CI

on:
  push:
    branches: [ main, 'feature/**' ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.24'
    
    - name: Run tests
      working-directory: src/refinery
      run: |
        go test ./... -v -cover
        go test -race ./...  # Race condition detection
    
    - name: Build Docker image
      run: docker build -t refinery:ci ./src/refinery
    
    - name: Start services
      run: docker-compose up -d
    
    - name: E2E test
      run: |
        sleep 30  # Wait for services
        ./test-digger-e2e.ps1
    
    - name: Cleanup
      run: docker-compose down
```

**Monitor CI**:
- Check GitHub Actions tab for workflow status
- Review test output for failures
- Fix any CI-specific issues (timeouts, dependencies)

#### 12. **Merge to Main**
Once CI passes on feature branch:

```bash
# Create pull request (GitHub CLI)
gh pr create \
  --title "Currency Refactor: Unit-Based Architecture" \
  --body "Implements single-queue unit storage, merkle branch hashes, and complete proof chain preservation.

## Changes
- Refinery: QueueManager with single unit queue
- Models: JouleTorqUnit, TokenTorqIngot with merkle hashes
- Mint: Ingot validation for 3600 units
- Tests: 95%+ coverage across all components

## Testing
- Unit tests: 96% coverage
- Integration tests: Full pipeline verified
- E2E test: Digger → Refinery → Mint flow working
- Performance: 300 units/sec throughput

## Breaking Changes
- Ore processing now creates 1 unit per token
- Ingots require exactly 3600 units (was variable joules)

## Migrations
None - fresh deployment

Closes #42" \
  --base main \
  --head feature/currency-refactor

# Wait for PR checks to pass
gh pr checks

# Merge (requires approval in production)
gh pr merge --squash --delete-branch
```

**Or via GitHub Web UI**:
1. Navigate to Pull Requests
2. Click "New Pull Request"
3. Select `feature/currency-refactor` → `main`
4. Add description (use template above)
5. Request reviews from team
6. Wait for approvals + CI green
7. Squash and merge

#### 13. **Push Main, Wait for CI**
Final validation on `main` branch:

```bash
# Pull merged changes
git checkout main
git pull origin main

# Verify version tag
git tag v1.0.0-currency-refactor
git push origin v1.0.0-currency-refactor

# Monitor CI on main
gh run watch
```

**Post-Merge Checklist**:
- ✅ CI passes on `main`
- ✅ Docker images built and tagged
- ✅ Documentation updated (README, architecture docs)
- ✅ Release notes published (if applicable)
- ✅ Stakeholders notified

---

## 📋 Quick Reference Checklists

### Starting New Feature
- [ ] Checkout feature branch: `git checkout -b feature/your-feature`
- [ ] Read relevant `ARCHITECTURE.md` files
- [ ] Review existing code patterns in target service
- [ ] Create implementation plan (TODO comments)
- [ ] Set up local testing environment: `docker-compose up -d`

### Before Each Commit
- [ ] Run unit tests: `go test ./... -v`
- [ ] Check coverage: `go test ./... -cover` (target: 95%+)
- [ ] Run integration tests (if applicable)
- [ ] Verify code compiles: `go build ./...`
- [ ] Check for race conditions: `go test -race ./...`
- [ ] Format code: `go fmt ./...`
- [ ] Lint: `golangci-lint run`

### Before Pushing
- [ ] All commits follow conventional format
- [ ] Run E2E test: `pwsh test-digger-e2e.ps1`
- [ ] Update architecture docs (if design changed)
- [ ] Update README (if public API changed)
- [ ] Update `docker-compose.yaml` (if config changed)
- [ ] Verify Docker build: `docker build .`
- [ ] Check for sensitive data (no secrets in code!)

### Before Merging
- [ ] CI passing on feature branch
- [ ] Code review approved (if team workflow)
- [ ] Documentation updated and reviewed
- [ ] Breaking changes documented
- [ ] Migration plan (if needed)
- [ ] Performance benchmarks (if critical path)

---

## 🔍 Code Review Guidelines

### What to Look For

**Architecture**:
- Does code follow service boundaries?
- Are dependencies injected (not global)?
- Is concurrency handled safely (mutexes, channels)?

**Testing**:
- Unit tests for all new functions?
- Coverage ≥95%?
- Edge cases tested (empty input, overflow, nil)?
- Integration tests for cross-component flows?

**Observability**:
- Structured logging with context?
- Prometheus metrics instrumented?
- Errors logged with stack traces?

**Documentation**:
- Architecture docs updated?
- Code comments explain WHY, not what?
- Public APIs documented?

**Performance**:
- No busy-waiting (use channels/conditions)?
- Bounded queues (no unbounded growth)?
- Graceful degradation under load?

### Review Comments Format

**Good**:
```
LGTM! QueueManager uses sync.Cond perfectly for blocking.

Minor: Consider adding a context.Context to GetUnit() for 
cancellation during shutdown. See Mint's IngotBuffer.Pop() 
for reference.
```

**Bad**:
```
This doesn't work.
```

---

## 🛠️ Common Patterns & Anti-Patterns

### ✅ DO: Dependency Injection
```go
// Good: Dependencies passed to constructor
type MintEngine struct {
    logger  *slog.Logger
    metrics *Metrics
    hasher  HashFunction
}

func NewMintEngine(logger *slog.Logger, metrics *Metrics, hasher HashFunction) *MintEngine {
    return &MintEngine{
        logger:  logger,
        metrics: metrics,
        hasher:  hasher,
    }
}
```

### ❌ DON'T: Global State
```go
// Bad: Global variables
var globalLogger *slog.Logger
var globalMetrics *Metrics

type MintEngine struct{}

func (me *MintEngine) Process() {
    globalLogger.Info("processing")  // Hard to test!
}
```

### ✅ DO: Context-Aware Blocking
```go
// Good: Respects cancellation
func (qm *QueueManager) GetUnit(ctx context.Context) (*JouleTorqUnit, error) {
    qm.mu.Lock()
    defer qm.mu.Unlock()
    
    for len(qm.units) == 0 {
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        default:
            qm.notEmpty.Wait()
        }
    }
    
    return qm.units[0], nil
}
```

### ❌ DON'T: Busy-Waiting
```go
// Bad: Burns CPU
func (qm *QueueManager) GetUnit() (*JouleTorqUnit, error) {
    for len(qm.units) == 0 {
        time.Sleep(10 * time.Millisecond)  // Wasteful!
    }
    return qm.units[0], nil
}
```

### ✅ DO: Structured Logging
```go
// Good: Contextual fields
slog.Info("ingot assembled",
    "ingot_id", ingot.ID,
    "units", len(ingot.Units),
    "joules", ingot.JouleTorqTotal,
    "contracts", ingot.ContractIDs)
```

### ❌ DON'T: Printf Debugging
```go
// Bad: Unstructured, no filtering
fmt.Printf("Ingot: %v\n", ingot)
```

### ✅ DO: Granular Metrics
```go
// Good: Specific counters
type Metrics struct {
    IngotsReceivedTotal    prometheus.Counter
    IngotsValidTotal       prometheus.Counter
    IngotsInvalidTotal     prometheus.Counter
    ProcessingLatency      prometheus.Histogram
}

m.IngotsReceivedTotal.Inc()
if err := validate(ingot); err != nil {
    m.IngotsInvalidTotal.Inc()
} else {
    m.IngotsValidTotal.Inc()
}
```

### ❌ DON'T: Sparse Metrics
```go
// Bad: Can't diagnose issues
type Metrics struct {
    RequestsTotal prometheus.Counter  // Too generic
}
```

---

## 🧪 Testing Philosophy

### Test Pyramid

```
         ┌─────────┐
         │   E2E   │  10% - Full pipeline, slow, comprehensive
         │ (Python)│       tests/e2e/*.py
         └─────────┘
       ┌─────────────┐
       │ Integration │  20% - Multi-service, moderate speed
       │  (Python)   │       tests/integration/*.py
       └─────────────┘
     ┌─────────────────┐
     │   Unit Tests    │  70% - Single function, fast, reliable
     │     (Go)        │       src/{service}/*_test.go
     └─────────────────┘
```

### Test Organization

**Structure**:
```
tests/
├── README.md              # Test documentation
├── e2e/                   # End-to-end tests (Python)
│   ├── phase2_complete.py # Digger → Refinery → Mint
│   ├── phase3_complete.py # Phase2Ingot → Merkle → RT Unit
│   └── full_pipeline.py   # Complete: Digger → DistoDam
├── integration/           # Integration tests (Python)
│   ├── refinery_mint.py   # Refinery ↔ Mint
│   └── mint_distodam.py   # Mint ↔ DistoDam
└── fixtures/              # Shared test helpers
    ├── __init__.py
    ├── helpers.py         # Utilities (colors, docker, etc.)
    └── sample_data.py     # Test data generators
```

**Why This Structure**:
- **Root stays clean**: No test files in repo root
- **Easy discovery**: All tests in `tests/` directory
- **Reusable code**: Shared fixtures eliminate duplication
- **Clear separation**: Unit (Go) vs Integration/E2E (Python)

### Unit Tests (Go) - 70% of coverage

**Location**: `src/{service}/internal/{component}/*_test.go`  
**Framework**: Go testing + testify/assert  
**Run**: `go test ./... -v -cover`

**Example**:
```go
// Test ONE function in isolation
func TestQueueManager_AddUnit(t *testing.T) {
    qm := NewQueueManager(10)
    
    unit := &models.JouleTorqUnit{TokenID: "test-1"}
    
    err := qm.AddUnit(unit)
    
    assert.NoError(t, err)
    assert.Equal(t, 1, qm.Len())
}

// Test concurrency safety
func TestQueueManager_ConcurrentAccess(t *testing.T) {
    qm := NewQueueManager(1000)
    
    // 10 goroutines adding units
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            unit := &models.JouleTorqUnit{
                TokenID: fmt.Sprintf("concurrent-%d", id),
            }
            qm.AddUnit(unit)
        }(i)
    }
    
    wg.Wait()
    assert.Equal(t, 10, qm.Len())
}
```

**Run Unit Tests**:
```bash
# All services
go test ./... -v -cover

# Specific service
cd src/mint
go test ./internal/mint -v -cover

# With race detection
go test ./... -v -race

# Coverage report
go test ./... -coverprofile=coverage.out
go tool cover -html=coverage.out
```

### Integration Tests (Python) - 20% of coverage

**Location**: `tests/integration/*.py`  
**Framework**: Python asyncio + nats-py  
**Run**: `python tests/integration/{test_name}.py`

**Example** (`tests/integration/refinery_mint.py`):
```python
#!/usr/bin/env python3
"""
Integration Test: Refinery → Mint
Tests ingot publishing and reception
"""

import asyncio
import json
from nats.aio.client import Client as NATS
import sys
sys.path.append('tests/fixtures')
from helpers import *

NATS_URL = "nats://localhost:4222"
INGOT_TOPIC = "mint.ingots"

async def test_refinery_to_mint():
    """Test that Mint receives and processes ingots from Refinery"""
    
    print_section("Integration Test: Refinery → Mint")
    
    # Connect to NATS
    nc = NATS()
    await nc.connect(NATS_URL)
    
    received_ingots = []
    
    async def ingot_handler(msg):
        ingot = json.loads(msg.data.decode())
        received_ingots.append(ingot)
        print_success(f"Received ingot: {ingot['id']}")
    
    # Subscribe to ingot topic
    await nc.subscribe(INGOT_TOPIC, cb=ingot_handler)
    
    # Publish test ingot
    test_ingot = create_sample_phase2_ingot("test-ingot-001")
    await nc.publish(INGOT_TOPIC, json.dumps(test_ingot).encode())
    
    # Wait for processing
    await asyncio.sleep(2)
    
    # Verify
    assert len(received_ingots) == 1, "Should receive 1 ingot"
    assert received_ingots[0]['id'] == "test-ingot-001"
    
    await nc.close()
    print_success("✨ Integration test passed!")

if __name__ == "__main__":
    asyncio.run(test_refinery_to_mint())
```

**Run Integration Tests**:
```bash
# Single test
python tests/integration/refinery_mint.py

# All integration tests (with pytest)
pytest tests/integration/ -v
```

### End-to-End Tests (Python) - 10% of coverage

**Location**: `tests/e2e/*.py`  
**Framework**: Python asyncio + nats-py + docker helpers  
**Run**: `python tests/e2e/{test_name}.py`

**Example** (`tests/e2e/phase3_complete.py`):
```python
#!/usr/bin/env python3
"""
E2E Test: Phase 3 Complete Proof Chain
Tests: Phase2Ingot → IngotHashQueue → Level2Merkle → Phase3Unit → DistoDam
"""

import asyncio
import json
from nats.aio.client import Client as NATS
import sys
sys.path.append('tests/fixtures')
from helpers import *

NATS_URL = "nats://localhost:4222"
INGOT_COUNT = 1000
TIMEOUT = 30

async def main():
    print_section("Phase 3 E2E Test: Complete Proof Chain")
    
    # Prerequisites
    if not check_docker_container("robotorq-network-mint-1"):
        print_error("Mint container not running")
        return False
    
    # Connect to NATS
    nc = NATS()
    await nc.connect(NATS_URL)
    
    received_units = []
    
    # Subscribe to output
    async def unit_handler(msg):
        unit = json.loads(msg.data.decode())
        received_units.append(unit)
        print_success(f"Received Phase3RoboTorqUnit: {unit['unit_id']}")
    
    await nc.subscribe("distodam.units", cb=unit_handler)
    await asyncio.sleep(1)  # Ensure subscription ready
    
    # Publish 1000 Phase2Ingots
    print_step(f"Publishing {INGOT_COUNT} Phase2Ingots...")
    for i in range(1, INGOT_COUNT + 1):
        ingot = create_sample_phase2_ingot(
            f"test-ingot-{i:04d}",
            contracts=["e2e-contract-001", "e2e-contract-002"],
            diggers=["e2e-digger-001"]
        )
        await nc.publish("mint.ingots", json.dumps(ingot).encode())
    
    print_success(f"Published {INGOT_COUNT} Phase2Ingots")
    
    # Wait for Phase3 unit
    print_step("Waiting for Phase3RoboTorqUnit...")
    await asyncio.sleep(TIMEOUT)
    
    # Verify
    if not received_units:
        print_error("No Phase3RoboTorqUnit received!")
        return False
    
    unit = received_units[0]
    is_valid, issues = verify_phase3_unit(unit)
    
    if not is_valid:
        print_error(f"Unit validation failed: {issues}")
        return False
    
    print_success("✨ PHASE 3 E2E TEST PASSED! ✨")
    await nc.close()
    return True

if __name__ == "__main__":
    success = asyncio.run(main())
    exit(0 if success else 1)
```

**Run E2E Tests**:
```bash
# Prerequisites
docker-compose up -d
pip install nats-py

# Single E2E test
python tests/e2e/phase3_complete.py

# All E2E tests
pytest tests/e2e/ -v
```

### Test Fixtures & Helpers

All shared test code lives in `tests/fixtures/`:

```python
# tests/fixtures/helpers.py
from helpers import (
    print_success,      # ✅ Green success message
    print_error,        # ❌ Red error message
    print_warning,      # ⚠️  Yellow warning
    print_step,         # ▶  Blue info step
    print_section,      # Section header with ===
    check_docker_container,      # Check if container running
    get_docker_logs,             # Get container logs
    search_docker_logs,          # Search logs for pattern
    wait_for_service,            # Wait for container healthy
    create_sample_phase2_ingot,  # Generate test ingots
    verify_phase3_unit,          # Validate Phase3 units
    validate_merkle_root,        # Check merkle hash format
)
```

**Usage in tests**:
```python
import sys
sys.path.append('tests/fixtures')
from helpers import *

# Now use helpers
print_step("Starting test...")
if check_docker_container("robotorq-network-mint-1"):
    print_success("Mint is running")
else:
    print_error("Mint not found")
```

---

## 🐛 Debugging Tests

### Debugging Python E2E Tests

**Check Prerequisites**:
```bash
# Verify services running
docker-compose ps

# Check specific service
docker ps | grep mint

# View service logs
docker logs robotorq-network-mint-1 --tail 50 -f
```

**Use Test Helpers**:
```python
# In your test file
import sys
sys.path.append('tests/fixtures')
from helpers import *

# Check if service is ready
if not check_docker_container("robotorq-network-mint-1"):
    print_error("Mint not running - start with: docker-compose up -d mint")
    exit(1)

# Wait for service to be healthy
if not wait_for_service("robotorq-network-mint-1", timeout=30):
    print_error("Mint failed to start")
    print_step("Check logs: docker logs robotorq-network-mint-1")
    exit(1)

# Search logs for specific events
logs = get_docker_logs("robotorq-network-mint-1", since="60s")
if "ingot assembled" in logs:
    print_success("Ingot processing confirmed")
else:
    print_warning("No ingots found in logs")
```

**Common Issues**:

1. **NATS Connection Refused**:
   ```python
   # Issue: nc.connect() fails
   # Solution: Check NATS is running
   if not check_docker_container("robotorq-network-nats-1"):
       print_error("NATS not running")
       exit(1)
   ```

2. **Test Timeout**:
   ```python
   # Issue: Waiting forever for message
   # Solution: Add timeout and check logs
   try:
       await asyncio.wait_for(
           asyncio.Future(),  # Your wait logic
           timeout=30
       )
   except asyncio.TimeoutError:
       print_error("Timeout waiting for Phase3Unit")
       logs = get_docker_logs("robotorq-network-mint-1", since="60s")
       print_step("Recent logs:")
       print(logs[-500:])  # Last 500 chars
   ```

3. **Invalid Test Data**:
   ```python
   # Issue: Unit validation fails
   # Solution: Use verify_* helpers
   is_valid, issues = verify_phase3_unit(unit)
   if not is_valid:
       print_error("Validation failed:")
       for issue in issues:
           print(f"  - {issue}")
   ```

### Debugging Go Unit Tests

**Run with Verbose**:
```bash
go test ./... -v -run TestSpecificTest
```

**Debug Race Conditions**:
```bash
go test ./... -race -run TestConcurrent
```

**Print Debug Info**:
```go
func TestDebug(t *testing.T) {
    unit := &JouleTorqUnit{TokenID: "debug"}
    t.Logf("Unit: %+v", unit)  // Prints during test
}
```

---

## 📊 Performance Benchmarks

### Running Benchmarks
```bash
# All benchmarks
go test -bench=. -benchmem ./internal/refinery

# Specific benchmark
go test -bench=BenchmarkQueueManager -benchmem ./internal/refinery
```

### Example Benchmark
```go
func BenchmarkQueueManager_Throughput(b *testing.B) {
    qm := NewQueueManager(100000)
    
    unit := &models.JouleTorqUnit{TokenID: "bench"}
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        qm.AddUnit(unit)
    }
}

// Output:
// BenchmarkQueueManager_Throughput-8   5000000   250 ns/op   64 B/op   1 allocs/op
//                                       ^^^^^     ^^^^^^^     ^^^^^^    ^^^^^^^^^^^
//                                       ops       ns/op       bytes     allocations
```

### Performance Targets
- **Refinery**: 300 units/sec sustained
- **Mint**: 1 batch/60s (1000 ingots)
- **Memory**: <500MB per service
- **Latency**: p99 < 100ms for API calls

---

## 🚀 Real-World Example: Currency Refactor

This workflow achieved **95% completion** in one sprint. Here's how:

### Week 1: Planning
- Read `README.md` Appendices (data structures, formulas)
- Diagrammed current flow (dual queues)
- Identified issue: Lost token granularity in aggregation
- Designed solution: Single unit queue with merkle trees

### Week 2: Refinery Refactor
```bash
# Day 1: QueueManager
git checkout -b feature/currency-refactor
# - Implement AddUnit/GetUnit
# - Write unit tests (100% coverage)
# - Commit: "feat(refinery): Add QueueManager single unit queue"

# Day 2: Unit Extraction
# - Update OreReceiver.extractUnits()
# - Distribute joules/robo per token
# - Commit: "feat(refinery): Extract JouleTorqUnits from ore"

# Day 3: Ingot Assembly
# - Refactor IngotAssembler to consume units
# - Commit: "refactor(refinery): Use unit-based ingot assembly"

# Day 4: Merkle Hashes
# - Implement CalculateBranchHash()
# - Commit: "feat(models): Add merkle branch hash to ingots"

# Day 5: Integration Test
# - Write ore → unit → ingot pipeline test
# - Commit: "test(refinery): Add integration test for unit flow"
```

### Week 3: Mint Integration
```bash
# Day 1: Ingot Validation
# - Update Mint to expect 3600 units
# - Reject ingots without units array
# - Commit: "feat(mint): Validate ingot unit count (3600)"

# Day 2: Merkle Tree Aggregation
# - Build batch hash from ingot hashes
# - Commit: "feat(mint): Aggregate ingot merkle hashes"

# Day 3-4: E2E Test
# - Implement test-digger-e2e.ps1
# - Verify full pipeline
# - Commit: "test(e2e): Add Digger → Refinery → Mint test"

# Day 5: Documentation
# - Rebuild MINT_ARCHITECTURE.md
# - Rebuild REFINERY_ARCHITECTURE.md
# - Commit: "docs: Rebuild architecture for currency refactor"
```

### Week 4: Polish & Merge
```bash
# Day 1: Remove outdated TODOs
# Day 2: CI/CD updates
# Day 3: PR review, address feedback
# Day 4: Merge to main
# Day 5: Monitor production deploy
```

**Result**: **95% complete**, only crypto signatures remain (~12h work).

---

## 🔐 Security Reminders

- **No secrets in code**: Use environment variables
- **Validate all inputs**: Never trust external data
- **Sanitize logs**: Don't log sensitive data (private keys, passwords)
- **Use prepared statements**: (When SQL added in future)
- **Principle of least privilege**: Services only access what they need

---

## 📚 Additional Resources

- **White Paper**: `README.md` (complete economic model)
- **Branching Strategy**: `BRANCHING.md`
- **Mint Architecture**: `src/mint/MINT_ARCHITECTURE.md`
- **Refinery Architecture**: `src/refinery/REFINERY_ARCHITECTURE.md`
- **Status Tracker**: `CURRENCY_REFACTOR_STATUS.md`
- **Prometheus Dashboard**: `Grafana/torq-observability-dashboard.json`

---

## 🎓 Final Tips

1. **Read the white paper first**: The economics drive the architecture
2. **Test as you build**: Don't write 1000 lines then test
3. **Commit frequently**: Small commits are easier to review and revert
4. **Document decisions**: Future you will thank present you
5. **Ask for help**: Architecture docs exist for a reason
6. **Measure everything**: Metrics reveal truth
7. **Respect context**: Graceful shutdown prevents data loss

**Remember**: This is a **monetary system based on physics**. Every line of code represents real energy, real computation, and real value. Code accordingly.

*"Watts > Wall Street"* 🤖⚡💰

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrokkingGrok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
