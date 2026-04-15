<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Hive Blockchain Bot Project Instructions

This is a Python bot for interacting with the Hive blockchain. When working on this project:

## Context
- This bot monitors Hive communities for posts with specific metadata
- It automatically replies to qualifying posts and sends HBD transfers
- Uses the `beem` library for Hive blockchain interactions
- Designed for deployment on Replit

## Code Style
- Follow Python PEP 8 guidelines
- Use type hints where appropriate
- Include comprehensive logging
- Handle errors gracefully
- Use environment variables for sensitive data

## Key Components
- `main.py`: Main bot logic and HiveCheckinBot class
- `config.json`: Bot configuration (non-sensitive settings)
- `.env`: Environment variables for sensitive data
- `requirements.txt`: Python dependencies

## Hive Blockchain Specifics
- Use `beem` library for Hive interactions
- Handle Hive account credentials securely
- Implement proper error handling for blockchain operations
- Consider rate limiting to avoid spam
- Use appropriate memo formats for transfers

## Testing Considerations
- Test with testnet before mainnet deployment
- Verify metadata checking logic thoroughly
- Test HBD transfer functionality carefully
- Monitor for duplicate post processing

## Security Best Practices
- Never hardcode private keys
- Use environment variables for credentials
- Implement proper key management
- Add rate limiting and spam protection

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/menobass) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
