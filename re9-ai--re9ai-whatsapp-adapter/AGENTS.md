# GitHub Copilot Instructions for re9ai-whatsapp-adapter

## Repository Overview

This repository contains **[PROJECT_TYPE]** for the re9.ai platform - a chat-first renovation and budgeting platform. [CUSTOMIZE THIS DESCRIPTION BASED ON PROJECT TYPE]

## Project Context

**re9.ai** is a renovation and budgeting platform that connects users with construction managers, builders, material sellers, and service providers through chat interfaces (WhatsApp, Telegram, web chat). The platform leverages AI to understand user needs and facilitate connections between all stakeholders in renovation projects.

## Repository Structure

### Core Files
- [LIST YOUR MAIN PROJECT FILES HERE]
- README.md - Project overview and setup instructions
- WIKI.md - Wiki submodule management guide

### Automation Scripts
- `scripts/` - Project management and automation scripts:
  - `setup-default-project-template.sh` - Initial project template setup
  - `update-wiki.sh` - Wiki submodule synchronization script
  - [ADD OTHER PROJECT-SPECIFIC SCRIPTS]

## Wiki Submodule Documentation Context

⚠️ **IMPORTANT**: This repository includes a `wiki/` submodule that contains comprehensive architectural and business documentation.

### Wiki Structure and Content
The `wiki/` directory is a **read-only submodule** that mirrors the GitHub wiki repository. It contains:

#### Architecture Documentation (`wiki/architecture/`)
- System architecture and design documents
- Component diagrams and technical specifications
- Infrastructure and deployment documentation
- Security and authentication specifications

#### Business Documentation (`wiki/business/`)
- Data model documentation with core entities and relationships
- User stories for all platform personas
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
[CUSTOMIZE BASED ON YOUR PROJECT TYPE - EXAMPLES:]
- **For Infrastructure**: Terraform, AWS services, Kubernetes
- **For Backend**: Node.js, TypeScript, PostgreSQL, Redis
- **For Frontend**: React, Next.js, TypeScript, Tailwind CSS
- **For Mobile**: React Native, Expo

### Development Environment
[CUSTOMIZE BASED ON YOUR PROJECT REQUIREMENTS]

## Key Architectural Principles

1. **Chat-First Design** - All components support multiple chat platforms (WhatsApp, Telegram)
2. **AI Integration** - Ready for AI/ML workloads and automation
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
4. **Consider the impact on AI/ML integration requirements**
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
# [ADD PROJECT-SPECIFIC SETUP COMMANDS]
# Example for various project types:

# For Node.js projects:
npm install
npm run dev

# For Python projects:
pip install -r requirements.txt
python manage.py runserver

# For Terraform projects:
terraform init
terraform plan
```

### Syncing Documentation
```bash
# Always sync wiki before starting work
./scripts/update-wiki.sh

# Check wiki status
git status wiki/
```

### [ADD OTHER PROJECT-SPECIFIC SCENARIOS]

## Security and Compliance Notes

- **Data Privacy**: LGPD (Brazilian GDPR) compliance required
- **Authentication**: Professional verification layer
- **Encryption**: At-rest and in-transit encryption mandatory
- **Access Control**: Principle of least privilege
- **Audit Trails**: Comprehensive logging and monitoring

## Integration Points

### Chat Platforms
- **WhatsApp Business API** - Primary communication channel
- **Telegram Bot API** - Secondary communication channel
- **Web Chat** - Browser-based communication

### AI/ML Services
- **Amazon Bedrock** - AI/ML capabilities
- **Natural Language Processing** - Chat understanding
- **Document Processing** - File analysis and extraction

### Payment Processing
- **Brazilian Payment Methods** - PIX, credit cards, bank transfers
- **International Payments** - Credit cards, PayPal
- **Compliance** - PCI DSS, financial regulations

Remember: When in doubt about business requirements or architectural decisions, always reference the comprehensive documentation in the `wiki/` submodule!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/re9-ai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/re9-ai)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
