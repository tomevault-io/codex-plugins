# GitHub Copilot Instructions for re9ai-bdi-calculation-service

## Repository Overview

This repository contains the **BDI Calculation Service** for the re9.ai platform - a chat-first renovation and budgeting platform that ensures compliance with Brazilian construction cost regulations.

The BDI Calculation Service handles Brazilian BDI (Basic Index of Construction Costs) compliance calculations, ensuring that all budget estimates and cost calculations adhere to Brazilian construction regulations and standards as defined by TCU (Federal Court of Auditors).

## Project Context

**re9.ai** is a renovation and budgeting platform that connects users with construction managers, builders, material sellers, and service providers through chat interfaces (WhatsApp, Telegram, web chat). The platform leverages AI to understand user needs and facilitate connections between all stakeholders in renovation projects.

For the Brazilian market, compliance with BDI calculations is critical for accurate budgeting and regulatory adherence.

## Repository Structure

### Core Files
- `src/` - Main source code for the BDI calculation service
- `calculations/` - BDI calculation algorithms and formulas
- `api/` - REST API endpoints for BDI services
- `data/` - BDI index data and regional variations
- `tests/` - Unit and integration tests
- README.md - Project overview and setup instructions
- WIKI.md - Wiki submodule management guide

### Automation Scripts
- `scripts/` - Project management and automation scripts:
  - `setup-default-project-template.sh` - Initial project template setup
  - `update-wiki.sh` - Wiki submodule synchronization script
  - `update-bdi-index.sh` - Script to update BDI index data from official sources
  - `validate-calculations.sh` - Script to validate calculation accuracy

## Wiki Submodule Documentation Context

⚠️ **IMPORTANT**: This repository includes a `wiki/` submodule that contains comprehensive architectural and business documentation.

### Wiki Structure and Content
The `wiki/` directory is a **read-only submodule** that mirrors the GitHub wiki repository. It contains:

#### Architecture Documentation (`wiki/architecture/`)
- System architecture and design documents for BDI services
- Component diagrams and technical specifications
- Infrastructure and deployment documentation
- Security and authentication specifications

#### Business Documentation (`wiki/business/`)
- Data model documentation with core entities and relationships
- User stories for all platform personas related to BDI features
- Business requirements and workflow specifications
- Integration and payment processing requirements

#### Root Level Documentation
- Project vision and storytelling documents
- Development guidelines and best practices

### Wiki Usage Guidelines for Copilot
1. **Always reference wiki content** when providing architectural context or business logic explanations
2. **Use wiki documentation** to understand the broader system design when suggesting changes
3. **Reference specific wiki files** when explaining how components support business requirements
4. **Consider chat integration requirements** when working on communication or API features

⚠️ **Wiki Editing Restrictions**: 
- The wiki submodule is **read-only** in this repository
- Wiki content should only be edited through GitHub's wiki interface or by working on the wiki repo directly
- Use `./scripts/update-wiki.sh` to sync latest wiki changes locally
- Never commit changes directly to files in the `wiki/` directory

## Technology Stack

### Core Technologies
- **For Backend**: Python with FastAPI, Node.js with Express
- **For Data Storage**: PostgreSQL, Redis for caching
- **For API**: RESTful API for integration with other platform services
- **For Infrastructure**: Docker, Kubernetes, AWS EKS

### Development Environment
- Python 3.9+
- Node.js 16+
- Docker for containerization
- Kubernetes for orchestration
- AWS CLI for cloud interactions

## Key Architectural Principles

1. **Regulatory Compliance First** - All calculations must adhere to Brazilian construction regulations
2. **Chat-First Design** - All components support multiple chat platforms (WhatsApp, Telegram)
3. **Microservices Architecture** - Modular, scalable component design
4. **Infrastructure as Code** - Everything defined declaratively
5. **Multi-Environment Support** - Dev, staging, production environments
6. **Security by Design** - Encryption, authentication, authorization
7. **Cost Optimization** - Efficient resource utilization

## Development Guidelines for Copilot

### When Suggesting Changes
1. **Always consider the chat integration requirements** from the wiki
2. **Reference the system architecture** documentation
3. **Ensure changes align with the platform strategy**
4. **Consider the impact on BDI compliance requirements**
5. **Maintain consistency with project patterns and conventions**

### When Working with Business Logic
1. **Reference relevant user stories** from `wiki/business/user-stories/`
2. **Use the data model documentation** to understand entity relationships
3. **Reference chat integration patterns** for communication requirements
4. **Consider payment processing requirements** for financial operations

### When Explaining Technical Concepts
1. **Always reference wiki documentation** for authoritative information
2. **Use specific wiki files** to support technical explanations
3. **Consider the broader platform context** when making recommendations
4. **Reference architecture decisions** documented in the wiki

## Common Development Scenarios

### Setting Up Development Environment
```bash
# Clone the repository
git clone https://github.com/re9-ai/re9ai-bdi-calculation-service.git
cd re9ai-bdi-calculation-service

# Install Python dependencies
pip install -r requirements.txt

# Install Node.js dependencies (if applicable)
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with appropriate values

# Run the service
python app.py
# or
npm start
```

### Syncing Documentation
```bash
# Always sync wiki before starting work
./scripts/update-wiki.sh

# Check wiki status
git status wiki/
```

### Updating BDI Index Data
```bash
# Update BDI index data from official sources
./scripts/update-bdi-index.sh

# Validate calculation accuracy
./scripts/validate-calculations.sh
```

### Testing BDI Calculations
```bash
# Run unit tests
pytest tests/unit

# Run integration tests
pytest tests/integration

# Run compliance validation tests
pytest tests/compliance
```

## Security and Compliance Notes

- **Data Privacy**: LGPD (Brazilian GDPR) compliance required
- **Authentication**: Professional verification layer
- **Encryption**: At-rest and in-transit encryption mandatory
- **Access Control**: Principle of least privilege
- **Audit Trails**: Comprehensive logging and monitoring
- **Regulatory Compliance**: TCU and ABNT standards adherence

## Integration Points

### Chat Platforms
- **WhatsApp Business API** - Primary communication channel
- **Telegram Bot API** - Secondary communication channel
- **Web Chat** - Browser-based communication

### AI/ML Services
- **Amazon Bedrock** - AI/ML capabilities for cost analysis
- **Natural Language Processing** - Chat understanding for budget requests
- **Document Processing** - File analysis and extraction for budget documents

### Payment Processing
- **Brazilian Payment Methods** - PIX, credit cards, bank transfers
- **International Payments** - Credit cards, PayPal
- **Compliance** - PCI DSS, financial regulations

### Brazilian Market Compliance
- **TCU Standards** - Federal Court of Auditors compliance
- **ABNT Specifications** - Brazilian technical standards
- **BDI Calculations** - Basic Index of Construction Costs
- **Regional Variations** - State-specific cost variations

Remember: When in doubt about business requirements or architectural decisions, always reference the comprehensive documentation in the `wiki/` submodule!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/re9-ai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/re9-ai)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
