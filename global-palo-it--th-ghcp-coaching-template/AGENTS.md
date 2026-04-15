# AI Contextual Instructions

This document provides guidelines for AI to understand the context and constraints when working with specific folders in this repository.

## ⚠️ Important Rules
- Each case identifier strictly limits operations to its designated folder ONLY
- Do not create or modify files in other folders when a specific case is mentioned
- If unsure about the context, ask for clarification before proceeding

## Folder Context Rules

1. **convert-app**
   - **Context Identifier**: "case1"
   - **Instructions**: 
     - When prompted with "case1", ONLY operate within the `convert-app` folder
     - ⚠️ All other folders are strictly off-limits when case1 is mentioned
     - All operations and references must be contained within this folder

2. **diagram-to-app**
   - **Context Identifier**: "case2"
   - **Instructions**: 
     - When prompted with "case2", ONLY operate within the `diagram-to-app` folder
     - ⚠️ All other folders are strictly off-limits when case2 is mentioned
     - All operations and references must be contained within this folder

3. **optimize-and-unittest**
   - **Context Identifier**: "case3"
   - **Instructions**: 
     - When prompted with "case3", ONLY operate within the `optimize-and-unittest` folder
     - ⚠️ All other folders are strictly off-limits when case3 is mentioned
     - All operations and references must be contained within this folder

4. **simple-api-system**
   - **Context Identifier**: "case4"
   - **Instructions**: 
     - When prompted with "case4", ONLY operate within the `simple-api-system` folder
     - ⚠️ All other folders are strictly off-limits when case4 is mentioned
     - All operations and references must be contained within this folder

## General Instructions

- The AI must strictly adhere to the context rules specified for each folder
- Any deviation from these rules should only occur if explicitly directed by the user
- If a case identifier is mentioned, work MUST be confined to that case's designated folder
- When in doubt, ask for clarification rather than making assumptions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GLOBAL-PALO-IT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
