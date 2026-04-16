# Roomshare Backend – AI Coding Agent Instructions

## Architecture Overview

**Roomshare** is a room-sharing booking platform with a Go backend and PostgreSQL database. The architecture follows a simple handler-model-database pattern:

```
main.go (server entry) 
  ↓
handlers/ (HTTP endpoints, business logic)
  ├── rooms.go (GET/POST rooms with capacity validation)
  └── booking.go (POST/GET bookings with occupancy checks)
  ↓
models/ (JSON request/response structs)
  ├── rooms.go
  └── booking.go
  ↓
db/ (PostgreSQL connection)
  └── postgres.go
```

### Key Design Pattern
- **Handler-per-entity**: `RoomHandler()` and `BookingHandler()` dispatch GET/POST using `switch r.Method`
- **Models act as DTOs**: No ORM; direct struct scanning from `sql.Rows.Scan()`
- **Validation in handlers**: Capacity checks happen in `CreateBooking()` by querying `rooms` + `room_occupants` JOIN

## Critical Workflows

### Running the Application
```bash
go run main.go
# Server listens on :8080 with CORS enabled for all origins
```

### Database Setup
- **Connection**: `db.Connect()` in main.go opens PostgreSQL on `localhost:5433`
- **Credentials**: Hard-coded in [db/postgres.go](db/postgres.go#L9) (user=postgres, password=root)
- **Database**: `roomshare_db`
- **Tables required**: `rooms`, `bookings`, `room_occupants` (inferred from queries)

### Testing Endpoints
- `GET /rooms` – fetch all rooms
- `POST /rooms` – create room (requires JSON body with `owner_id`, `location`, `price`, `capacity`, `description`)
- `GET /bookings` – fetch all bookings
- `POST /bookings` – create booking (requires `room_id`, `user_id`, `start_date`, `end_date`) with occupancy validation
- `GET /health` – liveness check

## Codebase Patterns

### Handler Pattern
All handlers follow this template (see [handlers/rooms.go](handlers/rooms.go#L11)):
```go
func GetRooms(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet { /* validate */ }
    // Query, scan into model slice, encode JSON
    json.NewEncoder(w).Encode(rooms)
}
```

**Never** mix multiple HTTP methods in a single function; use the dispatcher handler pattern.

### Booking Capacity Validation
Critical business logic in [handlers/booking.go](handlers/booking.go#L18-L43):
- Query **both** `rooms.capacity` and `COUNT(room_occupants.user_id)` in one statement
- Return `http.StatusConflict` (409) if `occupants >= capacity`
- Always check for `sql.ErrNoRows` when expecting a single result

### JSON Struct Tags
Use `json:"field_name"` for marshaling (see [models/booking.go](models/booking.go#L4)). **Note**: Booking model has malformed tags with spaces (`json: "id" `) – fix to `json:"id"` when refactoring.

## Integration Points

### Database Queries
- Use parameterized queries with `$1, $2, ...` placeholders (PostgreSQL syntax)
- Always defer `rows.Close()` for result sets
- Handle `sql.ErrNoRows` explicitly (e.g., "Room not found")

### CORS
Middleware in [main.go](main.go#L8) allows requests from any origin with GET/POST/OPTIONS.

### Frontend Integration
Frontend files in `frontend/` expect API at `http://localhost:8080`. Endpoints must match JavaScript calls in [frontend/js/api.js](frontend/js/api.js) (not yet reviewed).

## Development Notes

- **No tests yet**: Add unit tests for capacity validation logic
- **Hardcoded DB credentials**: Move to environment variables (`.env` file)
- **Future features**: User preference matching algorithm mentioned in [readme.md](readme.md)
- **Dependencies**: Only `github.com/lib/pq` for PostgreSQL driver; no additional frameworks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohammedMetlej) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
