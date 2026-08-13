# Copilot Instructions for Wix Client Profile Automation

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a Google Apps Script project for automating Wix client profile creation. The system integrates Google Sheets with Wix CMS to streamline client onboarding.

## Key Technologies
- Google Apps Script (JavaScript)
- Wix REST API
- Google Sheets API
- HTML/CSS for user interfaces
- Wix CMS Collections

## Project Structure
- `/src/` - Core Google Apps Script files
- `/config/` - Configuration and API credential templates
- `/ui/` - HTML/CSS files for user interfaces
- `/utils/` - Utility functions and helpers
- `/docs/` - Documentation and setup guides

## Coding Standards
- Use modern JavaScript (ES6+) where supported by Google Apps Script
- Follow Google Apps Script naming conventions
- Include comprehensive error handling for API calls
- Use descriptive variable and function names
- Add JSDoc comments for all functions
- Implement proper logging for debugging

## API Integration Guidelines
- All Wix API calls should include proper authentication
- Implement rate limiting and retry logic
- Use batch operations where possible for efficiency
- Handle API errors gracefully with user-friendly messages

## Google Sheets Integration
- Use SpreadsheetApp for sheet operations
- Implement data validation before processing
- Use named ranges for important data sections
- Provide clear feedback to users about processing status

## Security Considerations
- Store API credentials securely using PropertiesService
- Validate all user inputs
- Use HTTPS for all external API calls
- Implement proper error handling without exposing sensitive information

---
> Source: [zakpestsos/call-center-profiles](https://github.com/zakpestsos/call-center-profiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
