# Copilot Instructions for Ingombrante Discovery Question Engine

## Project Overview
This is an active demo for an adaptive questionnaire system designed for strategic insight gathering without intrusive data collection. Currently deployed at https://ingombrante.com/demos/. Built with extensibility in mind for deployment across multiple sectors (healthcare, finance, education, government, etc.).

## Architecture Goals
- **Privacy-First Design**: Minimize data collection while maximizing strategic insights
- **Sector Agnostic**: Modular design that adapts to different industries and use cases
- **Adaptive Flow**: Questions dynamically adjust based on response patterns and context
- **Non-Intrusive**: Avoid collecting personally identifiable information
- **Demo-Ready**: Focus on showcasing core capabilities while maintaining production-quality architecture
- **Extensible Foundation**: Build for easy customization and deployment across sectors

## Development Approach

### Core Components (To Be Implemented)
- **Question Engine**: Pluggable logic for selecting questions based on response patterns and sector context
- **Sector Adapters**: Modular components that customize question sets and logic for specific industries
- **Response Processor**: Analyzes answers to determine strategic insights with sector-specific interpretation
- **Adaptive Logic**: Algorithm that personalizes question flow based on user journey and sector requirements
- **Insight Generator**: Synthesizes responses into actionable recommendations tailored to sector context
- **Demo Scenarios**: Pre-built question flows showcasing capabilities across different use cases
- **Configuration Engine**: Allows easy customization of question sets, logic, and branding for different deployments

### Technology Stack Considerations
For maximum flexibility and demo appeal:
- **Frontend**: React/Vue.js with component library for consistent theming across sectors
- **Backend**: Node.js/Python with plugin architecture for sector-specific logic
- **Data**: In-memory processing with optional encrypted local storage for demo persistence
- **Configuration**: JSON/YAML-based sector configuration files for easy customization
- **Deployment**: Containerized for easy deployment across different environments
- **Analytics**: Pattern recognition with exportable insights dashboard for demo purposes

### Privacy-First Patterns
- Use session-based temporary storage, not persistent user profiles
- Implement data aggregation and anonymization at collection time
- Design questions to infer insights without asking direct personal questions
- Consider differential privacy techniques for any data analysis

### Question Design Patterns
- **Indirect Questioning**: Ask about scenarios rather than personal specifics
- **Hypothetical Framing**: "If your organization faced X, how would you prioritize Y?"
- **Comparative Analysis**: Present options rather than asking for personal details
- **Strategic Scenarios**: Focus on business decisions and organizational challenges
- **Sector-Specific Contexts**: Healthcare compliance vs. financial risk vs. educational outcomes
- **Progressive Disclosure**: Start general, become more specific based on sector and responses

### Development Workflow
Multi-track development approach for demo expansion, production readiness, and sector specialization:
1. Review current demo functionality at https://ingombrante.com/demos/ before making changes
2. Use feature branches for each development track to maintain demo stability
3. Implement modular architecture that supports both demo scenarios and production deployment
4. Build sector-specific features as plugins that can enhance both demo and production versions
5. Maintain continuous integration that protects the working demo while enabling rapid iteration
6. Test privacy audit mindset - what could be inferred vs. what's actually collected
7. Design for both immediate demo impact and long-term production scalability

### File Organization (Suggested)
```
src/
├── core/              # Core questionnaire engine (shared by demo & production)
├── demo/              # Demo-specific features and scenarios
├── production/        # Production-ready features (auth, multi-tenancy, APIs)
├── sectors/           # Sector-specific adapters and specializations
├── questions/         # Question definitions and templates
├── processors/        # Response analysis and insight generation
├── config/           # Configuration schemas and deployment settings
├── shared/           # Reusable components and utilities
└── tests/            # Comprehensive testing for all tracks
```

## Key Principles
- **Question over Collection**: Design questions that reveal insights without collecting sensitive data
- **Adaptive not Invasive**: Personalization through smart question selection, not data retention
- **Strategic Value**: Every question should contribute to actionable business insights
- **Privacy by Design**: Build privacy protection into the core architecture, not as an afterthought

## Testing Strategy
- Test question flows with diverse user scenarios
- Validate that insights are valuable without being intrusive
- Ensure no PII can be reconstructed from stored data
- Test adaptive logic with various response patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jason-fells-research)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jason-fells-research)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
