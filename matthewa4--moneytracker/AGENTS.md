# MoneyTracker Project - Development Instructions

## Project Overview
MoneyTracker is a C++ budgeting application that imports bank CSV files and generates professional Excel spreadsheets with budget analysis, categorization, and financial insights.

## Technology Stack
- **Language**: C++17
- **Build System**: CMake 3.12+
- **Compiler**: Ninja
- **Key Libraries**: Boost (system, filesystem, program_options), libxlsxwriter
- **Architecture**: Modular with clean separation of concerns

## Project Structure
```
MoneyTracker/
├── src/              # Implementation files
│   ├── main.cpp                    # CLI application
│   ├── CSVParser.cpp              # CSV parsing logic
│   ├── TransactionData.cpp        # Transaction data management
│   ├── BudgetAnalyzer.cpp         # Financial analysis
│   └── SpreadsheetGenerator.cpp   # Excel export
├── include/          # Header files
│   ├── CSVParser.h
│   ├── TransactionData.h
│   ├── BudgetAnalyzer.h
│   └── SpreadsheetGenerator.h
├── data/            # Sample CSV files
├── build/           # Build artifacts
├── CMakeLists.txt   # Build configuration
└── README.md        # User documentation
```

## Build Instructions

### Prerequisites
```bash
sudo apt-get install libboost-all-dev libxlsxwriter-dev cmake ninja-build
```

### Building
```bash
cd MoneyTracker
mkdir -p build
cd build
cmake -G Ninja ..
ninja
```

### Running
```bash
./moneytracker -i checking.csv -a "Checking" -o budget.xlsx --verbose
```

## Key Features Implemented

1. **CSV Parsing**
   - Auto-format detection (bank format vs generic)
   - Robust parsing with error handling
   - Transaction extraction with date, description, amount, balance

2. **Transaction Management**
   - Flexible transaction storage and querying
   - Category-based grouping
   - Monthly aggregation
   - Account separation

3. **Budget Analysis**
   - Income vs expense calculation
   - Spending by category breakdown
   - Monthly trend analysis
   - Top spending categories identification
   - Average calculations

4. **Excel Export**
   - Summary sheet with key metrics
   - Detailed transaction listing
   - Category breakdown
   - Monthly trends
   - Professional formatting with currencies

5. **CLI Interface**
   - Multiple file input support
   - Account naming
   - Format specification
   - Verbose mode for debugging
   - Help documentation

## API Design

### CSVParser
- `parseBank()` - Parse bank format (debit/credit columns)
- `parseGeneric()` - Parse generic CSV
- `parse()` - Auto-detect and parse

### TransactionData
- `addTransaction()` / `addTransactions()` - Add data
- `getTransactionsByCategory()` - Category filtering
- `getTransactionsByDateRange()` - Date range filtering
- `getCategoryTotals()` - Summary calculations
- `getMonthlyTotals()` - Monthly aggregation

### BudgetAnalyzer
- `analyzeBudget()` - Get complete budget summary
- `getTopSpendingCategories()` - Top categories
- `getMonthlyTrends()` - Monthly analysis
- `getSpendingTrend()` - Trend calculation

### SpreadsheetGenerator
- `generateSpreadsheet()` - Main export function
- Multiple sheet creation methods

## Testing
Sample CSV files provided in `data/`:
- `sample_bank.csv` - 28 sample transactions
- `sample_savings.csv` - 11 sample transactions

Test with:
```bash
./build/moneytracker -i data/sample_bank.csv -a "Bank" -i data/sample_savings.csv -a "Savings"
```

## Future Enhancements
1. Add chart generation in Excel sheets
2. Implement custom category rules
3. Add support for more bank formats
4. Create GUI interface
5. Add database storage
6. Implement budget vs actual comparison
7. Add recurring transaction detection

## Code Quality
- C++17 standard compliance
- Header-only error handling
- No external dependencies beyond specified libraries
- Memory-efficient transaction processing
- Comprehensive error checking

## Build Statistics
- 5 implementation files
- 4 header files
- ~1100 lines of code
- Successfully compiles with no errors
- Only minor warnings on unused parameters

---
> Source: [MatthewA4/MoneyTracker](https://github.com/MatthewA4/MoneyTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
