# GitHub Copilot Instructions for MoneyS3

## Project Overview
MoneyS3 is a PHP library for generating XML documents compatible with Money S3 accounting software. It provides a fluent API for creating invoices, receipts, and other accounting documents with proper XML serialization.

**Key Technologies:**
- PHP 8.3+
- XMLWriter extension
- PHPStan (Level 8 static analysis)
- PHPUnit testing
- Docker development environment

## Architecture & Design Patterns

### Core Classes
- `MoneyS3`: Main entry point for the library
- `MoneyS3Data`: Internal data container with invoices and receipts collections
- `Element<T>`: Generic XML element wrapper with type safety
- `ISerializable`: Interface for XML serialization
- `IDocument`: Interface for document types (invoices, receipts)

### Design Patterns Used
- **Fluent Interface**: All setters return `self` for method chaining
- **Template/Generics**: `Element<T>` class supports typed values (string, int, float, bool, DateTime, objects)
- **Factory Pattern**: Document creation through `addInvoice()` and `addReceipt()`
- **Serialization**: Consistent XML output through `ISerializable` interface

## Development Environment

### Installation & Setup
```bash
# Clone and setup
git clone <repository>
cd MoneyS3

# Start Docker environment
make up

# Install dependencies
make install

# Alternative: manual Docker commands
cd docker && docker compose up --build -d
docker exec -it eproduct-moneys3 composer install
```

### Available Make Commands
- `make up` - Start Docker containers
- `make down` - Stop Docker containers  
- `make install` - Install Composer dependencies
- `make update` - Update Composer dependencies
- `make test` - Run PHPUnit tests
- `make ps` or `make phpstan` - Run PHPStan static analysis

### Docker Environment
- **Container**: `eproduct-moneys3`
- **PHP Version**: 8.3-fpm
- **Working Directory**: `/application`
- **Port**: 9000 (PHP-FPM)

## Testing

### Running Tests
```bash
# Run all tests
make test

# Manual PHPUnit execution
docker exec -it eproduct-moneys3 vendor/bin/phpunit --colors=always --testdox tests

# Run specific test class
docker exec -it eproduct-moneys3 vendor/bin/phpunit tests/Document/Invoice/InvoiceTest.php
```

### Test Structure
- **Unit Tests**: Individual class functionality (`tests/`)
- **Integration Tests**: End-to-end XML generation (`tests/Integration/`)
- **Utility Tests**: XML comparison and diff utilities (`tests/Utility/`)

### Test Utilities
- `XmlTestUtility`: Comprehensive XML comparison and diff functionality
- Custom assertions for XML structure validation
- XML content assertions for generated output

## Static Analysis

### PHPStan Configuration
- **Level**: 8 (strictest)
- **Baseline**: `phpstan-baseline.neon` (legacy issues)
- **Coverage**: `src/` directory
- **Parallel Processing**: Disabled for Docker compatibility

### Running PHPStan
```bash
# Quick analysis
make ps

# Full analysis with output
docker exec -it eproduct-moneys3 vendor/bin/phpstan analyse -c phpstan.neon src tests
```

### Type Safety Guidelines
- Use strict typing with `Element<T>` generics
- Prefer typed parameters over mixed/any types
- Add null checks for DOM operations
- Use `@phpstan-ignore-line` sparingly for legitimate edge cases

## Coding Guidelines

### XML Element Patterns
```php
// Create typed elements
$element = new Element<string>("ElementName", required: true);
$element->setValue("value");

// Fluent interface for documents
$invoice = $moneyS3->addInvoice(InvoiceType::ISSUED)
    ->setDocumentNumber('INV-001')
    ->setDescription('Test Invoice')
    ->setTotal(1000.00)
    ->setIssued(new DateTime('2023-01-01'));
```

### Type Declarations
```php
// Use specific types for Element generics
/** @var Element<string> */
private Element $documentNumber;

/** @var Element<float> */  
private Element $totalAmount;

/** @var Element<DateTime> */
private Element $issueDate;

/** @var Element<bool> */
private Element $creditNote;
```

### Serialization Patterns
```php
public function serialize(XMLWriter $writer): void
{
    // Always check required elements
    if ($this->requiredField === null && $this->isRequired) {
        throw new MoneyS3Exception("Required field not set");
    }
    
    // Serialize child elements
    $this->element->serialize($writer);
}
```

## Common Development Tasks

### Adding New Document Types
1. Create class implementing `IDocument`
2. Add serialization logic with XMLWriter
3. Create corresponding test class
4. Add factory method to `MoneyS3` class
5. Update `MoneyS3Data` collection

### Adding New Element Types
1. Define typed `Element<T>` properties
2. Initialize in constructor with XML element names
3. Create setter methods with fluent interface
4. Add validation for required fields
5. Implement serialization order

### XML Structure Validation
- Use `XmlTestUtility::compareXml()` for full comparison
- Use `XmlTestUtility::assertXmlContains()` for partial validation
- Test both structure and content separately
- Validate against Money S3 XML schema requirements

## Debugging & Troubleshooting

### Common Issues
- **Type Errors**: Check `Element<T>` generic types match usage
- **XML Validation**: Use `XmlTestUtility` for structure comparison
- **Docker Issues**: Restart containers with `make down && make up`
- **Dependencies**: Run `make install` after composer.json changes

### Debug XML Output
```php
$xml = $moneyS3->getXml();
echo $xml; // Raw XML output

// Or in tests
$this->assertStringContainsString('<ExpectedElement>', $xml);
```

### PHPStan Debugging
- Check `phpstan-baseline.neon` for suppressed issues
- Use `@phpstan-ignore-line` for legitimate false positives
- Run with `-v` flag for verbose output

## Best Practices

### Code Quality
- Maintain PHPStan level 8 compliance
- Write comprehensive tests for XML generation
- Use type hints consistently
- Follow PSR-4 autoloading standards

### XML Generation
- Always validate required fields before serialization
- Use consistent element naming (Czech Money S3 format)
- Handle null values gracefully in serialization
- Test XML output against expected structure

### Performance
- Use XMLWriter for efficient XML generation
- Minimize object creation in serialization loops
- Consider memory usage for large document collections

This codebase emphasizes type safety, clean XML generation, and comprehensive testing. Always run both tests and static analysis before committing changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ePREHLED-cz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ePREHLED-cz)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
