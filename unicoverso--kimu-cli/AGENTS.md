# KIMU-CLI Copilot & AI Agent Integration Guide

> **Agent Training Note:**
> To generate correct, robust, and maintainable code, you MUST read and follow not only the main guidelines below, but also all subchapters such as **Practical Examples**, **FAQ & Troubleshooting**, **Testing & Quality Assurance**, **Advanced Usage**, **Glossary**, and **CLI Commands Reference**. These sections contain essential rules, conventions, edge cases, and best practices. Always consult them when generating or modifying code.

This document provides direct, actionable instructions for Copilot and AI agents to support development and extension of the kimu-cli tool. All content is specific to kimu-cli and the KIMU ecosystem.

## How Copilot & AI Agents Should Use This File
> **Agent Training Note:** This file is your primary reference for generating code, understanding conventions, and following best practices in kimu-cli. It contains essential guidelines, examples, and patterns that you must follow to ensure consistency, maintainability, and correctness in all generated code.

- **Use English** for all code comments, documentation, and metadata.
- **Read the entire document** to understand the project structure, conventions, and best practices for kimu-cli.
- **Use the provided examples** as templates for creating new commands and utilities.
- **Follow the coding conventions** outlined in `CODE_GUIDELINES.md` and `CONTRIBUTING.md` and best practices described here.
- **Refer to the Practical Examples** section for implementation patterns and CLI best practices.
- **Follow the guidelines** for command creation, module management, and project scaffolding.
- **Use the FAQ & Troubleshooting** section for common issues and edge cases.
- **Propose changes** to this file if you identify missing patterns, conventions or best practices.
- **Use the Glossary** for precise definitions of key concepts and terminology.
- **Use the CLI Commands Reference** to understand command structure and argument patterns.
- **Use concise, readable code** and clear documentation in all generated code.
- **Prefer modularity and reusability** in all commands and utilities.

## Project Overview

### **What is KIMU-CLI**
KIMU-CLI is the official command-line interface for the KIMU framework ecosystem. It provides a unified tool for:
- Creating new KIMU applications from templates
- Installing and managing modules from kimu-modules repository
- Installing and managing extensions
- Building and serving KIMU applications
- Project diagnostics and health checks
- Ecosystem discovery and exploration

### **Role in KIMU Ecosystem**
```
KIMU Ecosystem Architecture:
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   kimu-core     │    │   kimu-modules   │    │ kimu-extensions │
│   (Framework)   │    │   (Optional      │    │ (Community      │
│                 │    │    Modules)      │    │  Extensions)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────▼──────────────┐
                    │         kimu-cli           │
                    │    (Developer Tool)        │
                    │                            │
                    │ • Project creation         │
                    │ • Module management        │
                    │ • Build orchestration      │
                    │ • Ecosystem integration    │
                    └────────────────────────────┘
```

### **Design Philosophy**
- **Simple**: Easy commands that "just work"
- **Consistent**: Unified pattern across all operations
- **Discoverable**: Self-documenting with helpful output
- **Modular**: Extensible architecture for future features
- **Robust**: Proper error handling and recovery
- **Fast**: Optimized for developer productivity

## KIMU Framework Overview

### **What is KIMU Framework**
KIMU (Keep It Minimal UI) is a lightweight, modular web framework focused on:
- **Minimal footprint**: Small bundle sizes and fast loading
- **Web Components**: Standards-based custom elements
- **TypeScript-first**: Type safety and developer experience
- **Modular architecture**: Install only what you need
- **Developer productivity**: Simple APIs and clear patterns

### **KIMU Ecosystem Components**

#### **kimu-core** (Framework Foundation)
```typescript
// The base framework providing:
- KimuComponentElement (base class for components)
- KimuEngine (rendering and lifecycle management)
- KimuApp (application container)
- Core utilities and types
- Extension loading system
```

**Purpose**: Provides the minimal foundation for KIMU applications
**Size**: ~8-12k minified (core only)
**Contains**: Essential framework APIs, no optional features

#### **kimu-modules** (Optional Functionality)
```typescript
// Separate repository with optional modules:
- router (routing system)
- i18n (internationalization)
- analytics (event tracking)
- storage (local storage management)
- auth (authentication)
```

**Purpose**: Provides optional functionality that can be added as needed
**Installation**: Via kimu-cli commands
**Dependency**: Each module may depend on others

#### **kimu-extensions** (UI Components & Features)
```typescript
// Reusable UI components and complete features:
- kimu-dashboard (admin interface)
- kimu-calendar (calendar component)
- kimu-chat (chat interface)
- Community extensions
```

**Purpose**: Ready-to-use UI components and application features
**Installation**: Via kimu-cli commands
**Customization**: Configurable and themeable

## KIMU-CLI Integration with KIMU-Core

### **How kimu-cli Interacts with kimu-core**

#### **Project Creation Workflow**
```bash
kimu create my-app
```

**What happens internally:**
1. **Downloads kimu-core**: Fetches the latest compatible version
2. **Creates project structure**: Sets up src/, dist/, configuration files
3. **Installs dependencies**: NPM packages and kimu-core itself
4. **Generates kimu.config.json**: Project configuration file
5. **Sets up build system**: Vite/Webpack configuration for KIMU
6. **Creates entry files**: main.ts with KIMU application setup

#### **Template System Integration**
```typescript
// Templates must be compatible with kimu-core architecture
interface KimuProjectTemplate {
  name: string;
  kimuCoreVersion: string;        // Required kimu-core version
  defaultModules: string[];       // Modules included by default
  structure: ProjectStructure;    // File/folder structure
  scripts: PackageScripts;        // NPM scripts for development
}

// Example: Dashboard template
{
  name: 'dashboard',
  kimuCoreVersion: '^1.0.0',
  defaultModules: ['router', 'analytics', 'auth'],
  structure: {
    'src/pages/': ['dashboard.ts', 'login.ts'],
    'src/components/': ['nav.ts', 'sidebar.ts']
  }
}
```

#### **Module Installation Integration**
```typescript
// When installing modules, kimu-cli must:
1. Check kimu-core compatibility
2. Resolve module dependencies
3. Install in correct project structure
4. Update build configuration
5. Generate type definitions
6. Update kimu.config.json

// Example module installation
async function installModule(moduleName: string) {
  // 1. Validate kimu-core version
  const coreVersion = await getKimuCoreVersion();
  const moduleInfo = await getModuleInfo(moduleName);
  
  if (!isCompatible(coreVersion, moduleInfo.kimuCore)) {
    throw new Error(`Module ${moduleName} requires kimu-core ${moduleInfo.kimuCore}, but found ${coreVersion}`);
  }
  
  // 2. Install in kimu-core structure
  await downloadModule(moduleName, 'src/modules/');
  
  // 3. Update build configuration
  await updateBuildConfig(moduleName);
}
```

### **Version Compatibility Management**

#### **Compatibility Matrix**
```typescript
interface CompatibilityMatrix {
  'kimu-cli': {
    '1.0.x': {
      'kimu-core': '^1.0.0',
      'node': '>=18.0.0',
      'npm': '>=8.0.0'
    },
    '1.1.x': {
      'kimu-core': '^1.0.0 || ^1.1.0 || ^1.2.0',
      'node': '>=18.0.0',
      'npm': '>=8.0.0'
    }
  }
}
```

#### **Version Checking Strategy**
```typescript
// kimu-cli must always verify compatibility before operations
async function checkCompatibility(projectPath: string): Promise<CompatibilityReport> {
  const kimuConfig = await readKimuConfig(projectPath);
  const packageJson = await readPackageJson(projectPath);
  
  return {
    kimuCore: {
      installed: packageJson.dependencies['@kimu/core'],
      required: kimuConfig.kimuCoreVersion,
      compatible: semver.satisfies(installed, required)
    },
    modules: await checkModuleCompatibility(kimuConfig.modules),
    node: await checkNodeVersion(),
    cli: await checkCliVersion()
  };
}
```

### **Build System Integration**

#### **kimu-cli Build Commands**
```bash
kimu build                    # Production build with kimu-core
kimu dev                      # Development server with HMR
kimu serve                    # Serve built application
```

#### **Build Configuration Generation**
```typescript
// kimu-cli generates build config compatible with kimu-core
interface KimuBuildConfig {
  entry: string;                    // main.ts with KIMU setup
  framework: 'kimu-core';           // Framework identifier
  modules: string[];                // Installed modules to include
  extensions: string[];             // Installed extensions to include
  optimization: {
    splitChunks: boolean;           // Separate chunks for modules
    treeshaking: boolean;           // Remove unused code
    minify: boolean;                // Minification settings
  };
  output: {
    format: 'esm' | 'iife';        // Output format
    globals: Record<string, string>; // Global variables
  };
}
```

### **Project Structure Generated by kimu-cli**

#### **Standard KIMU Project Structure**
```
my-kimu-app/                    # Created by: kimu create my-kimu-app
├── src/
│   ├── main.ts                 # KIMU application entry point
│   ├── modules/                # Modules installed via: kimu install module <name>
│   │   ├── router/             # Downloaded from kimu-modules
│   │   ├── i18n/               # Downloaded from kimu-modules
│   │   └── analytics/          # Downloaded from kimu-modules
┌   ├── extensions/             # Extensions installed via: kimu install extension <name>
┌   ├   ├── app-root/          # Default extension from kimu-core
│   │   └── kimu-dashboard/     # Downloaded from kimu-modules
│   ├── components/             # Custom project components
│   ├── assets/                 # Static assets
│   └── styles/                 # Global styles
├── dist/                       # Build output (generated by: kimu build)
├── kimu.config.json           # KIMU project configuration
├── package.json               # NPM configuration with kimu-core dependency
├── tsconfig.json              # TypeScript configuration for KIMU
├── vite.config.ts             # Build tool configuration
└── README.md                  # Generated documentation
```

#### **Generated main.ts Structure**
```typescript
// Generated by kimu-cli create command
import { KimuApp } from '@kimu/core';
import { KimuExtensionManager } from '@kimu/core';

// Import installed modules (added by kimu install module commands)
import RouterModule from './modules/router/module';
import I18nModule from './modules/i18n/module';
import AnalyticsModule from './modules/analytics/module';

async function main() {
  // Initialize KIMU application
  const app = await KimuApp.getInstance();
  
  // Initialize modules (based on kimu.config.json)
  const router = new RouterModule();
  const i18n = new I18nModule();
  const analytics = new AnalyticsModule();
  
  // Initialize extension manager
  const extensionManager = KimuExtensionManager.getInstance();
  await extensionManager.init();
  
  // Load main extension (configurable in kimu.config.json)
  await extensionManager.load('app-root');
  
  // Mount application
  const root = document.getElementById('app');
  const mainComponent = document.createElement('app-root');
  root?.appendChild(mainComponent);
}

main().catch(console.error);
```

### **Configuration Management**

#### **kimu.config.json Structure**
```json
{
  "name": "my-kimu-app",
  "version": "1.0.0",
  "kimuCore": "^1.0.0",
  "template": "basic",
  "modules": {
    "installed": ["router", "i18n", "analytics"],
    "registry": "https://github.com/unicoverso/kimu-modules",
    "dependencies": {
      "router": "^1.2.0",
      "i18n": "^1.5.0", 
      "analytics": "^0.8.0"
    }
  },
  "extensions": {
    "installed": ["app-root", "kimu-dashboard"],
    "main": "app-root",
    "dependencies": {
      "kimu-dashboard": "^2.1.0"
    }
  },
  "build": {
    "target": "es2020",
    "format": "esm",
    "sourcemap": true,
    "minify": true
  },
  "development": {
    "port": 3000,
    "host": "localhost",
    "open": true
  }
}
```

## Project Structure

Organize the kimu-cli project as follows:

```
kimu-cli/
│
├── src/                  # Main source code
│   ├── commands/         # CLI command implementations
│   │   ├── create.ts     # Project creation (kimu create)
│   │   ├── install.ts    # Module/extension installation
│   │   ├── list.ts       # Discovery commands
│   │   ├── remove.ts     # Removal commands
│   │   ├── update.ts     # Update commands
│   │   ├── build.ts      # Build and serve commands
│   │   └── info.ts       # Project info and diagnostics
│   ├── utils/            # Utility functions and helpers
│   │   ├── registry.ts   # kimu-modules integration
│   │   ├── dependency-resolver.ts # Dependency management
│   │   ├── project-detector.ts    # Project type detection
│   │   ├── template-manager.ts    # Template handling
│   │   ├── git-utils.ts          # Git operations
│   │   ├── file-utils.ts         # File system operations
│   │   └── logger.ts             # Consistent logging
│   ├── types/            # TypeScript type definitions
│   │   ├── cli-types.ts  # CLI-specific types
│   │   ├── registry-types.ts     # Registry and module types
│   │   └── project-types.ts      # Project configuration types
│   ├── templates/        # Project templates
│   │   ├── basic/        # Basic KIMU application
│   │   ├── dashboard/    # Dashboard application template
│   │   └── chat/         # Chat application template
│   └── config/           # Configuration and constants
│       ├── constants.ts  # Global constants
│       └── defaults.ts   # Default configurations
│
├── bin/                  # CLI entry points
│   └── kimu.js           # Main CLI executable
│
├── templates/            # Additional template resources
│   ├── project/          # Full project templates
│   └── component/        # Component generation templates
│
├── tests/                # Unit and integration tests
│   ├── commands/         # Command-specific tests
│   ├── utils/            # Utility function tests
│   └── integration/      # End-to-end tests
│
├── docs/                 # CLI documentation
│   ├── commands/         # Command-specific documentation
│   └── guides/           # Usage guides and tutorials
│
├── .github/              # GitHub workflows and configurations
│   ├── copilot-instructions.md   # This guide
│   └── workflows/        # CI/CD workflows
│
├── package.json          # NPM configuration with CLI bin
├── tsconfig.json         # TypeScript configuration
├── .env.example          # Environment variables template
├── README.md             # Project overview and quick start
├── CODE_GUIDELINES.md    # Coding conventions and style rules
├── CONTRIBUTING.md       # Contribution guidelines
├── SECURITY.md           # Security best practices
└── CHANGELOG.md          # Version history and changes
```

## Development Guidelines

For details and advanced usage of CLI patterns and APIs, refer to the Practical Examples and FAQ sections below.

### **Core Principles**
- Use TypeScript for all source code with strict mode enabled
- Follow commander.js patterns for command structure
- Implement proper error handling with user-friendly messages
- Use chalk for consistent colored output
- Implement loading indicators with ora for long operations
- Use inquirer for interactive prompts when needed
- Follow semantic versioning for all releases
- Write comprehensive tests for all commands and utilities
- Document all public APIs and command usage
- Use clear, concise help text for all commands
- Implement proper logging with different verbosity levels

### **Command Structure Patterns**
- Each command should be in its own file in `/src/commands/`
- Use consistent argument and option naming across commands
- Implement `--help` and `--version` flags for all commands
- Use consistent exit codes (0 for success, 1 for errors)
- Implement proper validation for all user inputs
- Use consistent output formatting with colors and symbols
- Implement dry-run mode where applicable (`--dry-run`)
- Support verbose mode for debugging (`--verbose`)

### **Registry Integration**
- Always use the official kimu-modules registry as primary source
- Implement fallback mechanisms for network failures
- Cache registry data locally for performance
- Support custom registry URLs via configuration
- Implement proper version resolution and compatibility checking
- Use semantic versioning for all module dependencies
- Support both stable and prerelease versions

### **Project Management**
- Detect KIMU projects using kimu.config.json presence
- Validate project structure before performing operations
- Maintain project configuration in kimu.config.json
- Support multiple KIMU project types (basic, dashboard, chat, etc.)
- Implement proper backup mechanisms for critical operations
- Use git integration where appropriate for tracking changes

## Getting Started
1. Clone the repository from GitHub:
   ```sh
   git clone https://github.com/UnicoVerso/kimu-cli.git
   ```
2. Run `npm install` to install dependencies.
3. Use `npm run build` to compile TypeScript.
4. Use `npm link` for local development and testing.
5. Run `kimu --help` to verify installation.
6. Review `/docs/` for detailed usage documentation.

## Installation & Configuration
- Node.js >= 18 required for development and usage.
- Global installation: `npm install -g kimu`
- Local installation in projects: `npm install --save-dev kimu`
- Configuration via kimu.config.json in project root
- Environment variables for advanced configuration

## CLI Commands & Scripts
- `npm run build`: Compile TypeScript to JavaScript
- `npm run test`: Run all tests with Jest
- `npm run lint`: Run ESLint for code quality
- `npm run dev`: Development mode with file watching
- `npm run link`: Link for local testing
- `npm run release`: Prepare and publish new release

## Build & Deployment
- Use `npm run build` for production builds.
- Output is in `/dist/` directory.
- NPM package published with CLI binary.
- Support for both global and local installation.

## Security Best Practices (Condensed)
- Validate all user inputs and file paths
- Sanitize downloaded content from external sources
- Use secure methods for file system operations
- Never execute arbitrary code from external sources
- Implement proper permission checks for file operations
- Review and follow `SECURITY.md` guidelines

## CLI Commands Reference

### **Project Creation**
```bash
kimu create <name>                 # Create new KIMU project
kimu create <name> --template <t>  # Use specific template
kimu create                        # Interactive mode
```

### **Module Management**
```bash
kimu install module <name>         # Install module from registry
kimu install <name>                # Auto-detect type (module/extension)
kimu remove <name>                 # Remove installed module/extension
kimu list modules                  # List available modules
kimu list installed               # List installed modules/extensions
```

### **Project Operations**
```bash
kimu build                         # Build current project
kimu dev                           # Start development server
kimu serve                         # Serve built project
kimu update                        # Update all installed modules
```

### **Information & Diagnostics**
```bash
kimu info                          # Show project information
kimu check                         # Check project health
kimu doctor                        # Comprehensive diagnostics
kimu version                       # Show CLI version
```

## Command Implementation Guidelines

### **Command Structure Template**
```typescript
import { Command } from 'commander';
import chalk from 'chalk';
import ora from 'ora';
import { Logger } from '../utils/logger';

export function setupCreateCommand(program: Command): void {
  program
    .command('create <name>')
    .description('Create a new KIMU application')
    .option('-t, --template <template>', 'project template', 'basic')
    .option('--no-install', 'skip npm install')
    .option('--verbose', 'verbose output')
    .action(async (name: string, options: CreateOptions) => {
      const logger = new Logger(options.verbose);
      const spinner = ora('Creating KIMU project...').start();
      
      try {
        await createProject(name, options, logger);
        spinner.succeed(chalk.green(`✅ Project ${name} created successfully!`));
      } catch (error) {
        spinner.fail(chalk.red(`❌ Failed to create project: ${error.message}`));
        process.exit(1);
      }
    });
}
```

### **Error Handling Patterns**
```typescript
// Always provide helpful error messages
try {
  await riskyOperation();
} catch (error) {
  if (error.code === 'ENOENT') {
    logger.error('File not found. Please check the path and try again.');
  } else if (error.code === 'EACCES') {
    logger.error('Permission denied. Try running with sudo or check file permissions.');
  } else {
    logger.error(`Unexpected error: ${error.message}`);
  }
  process.exit(1);
}
```

### **Registry Integration Patterns**
```typescript
// Always implement retry logic and caching
async function fetchModuleCatalog(): Promise<ModuleCatalog> {
  const cacheFile = path.join(getCacheDir(), 'catalog.json');
  
  try {
    // Try cache first
    if (await isCacheValid(cacheFile)) {
      return JSON.parse(await fs.readFile(cacheFile, 'utf8'));
    }
    
    // Fetch from registry with retry
    const catalog = await retryFetch(REGISTRY_URL, { retries: 3 });
    
    // Cache for future use
    await fs.writeFile(cacheFile, JSON.stringify(catalog, null, 2));
    
    return catalog;
  } catch (error) {
    // Fallback to cache if available
    if (await fs.pathExists(cacheFile)) {
      logger.warn('Using cached catalog due to network error');
      return JSON.parse(await fs.readFile(cacheFile, 'utf8'));
    }
    throw error;
  }
}
```

## Practical Examples
> **Agent Reminder:** Use these examples as reference for implementation patterns and CLI best practices.

### **Example: Create Command Implementation**
```typescript
// src/commands/create.ts
import { Command } from 'commander';
import { createProject } from '../utils/project-creator';
import { validateProjectName } from '../utils/validators';
import { Logger } from '../utils/logger';

interface CreateOptions {
  template: string;
  install: boolean;
  verbose: boolean;
}

export function setupCreateCommand(program: Command): void {
  program
    .command('create <name>')
    .description('Create a new KIMU application')
    .option('-t, --template <template>', 'project template to use', 'basic')
    .option('--no-install', 'skip automatic npm install')
    .option('--verbose', 'enable verbose output')
    .action(async (name: string, options: CreateOptions) => {
      const logger = new Logger(options.verbose);
      
      // Validate input
      if (!validateProjectName(name)) {
        logger.error('Invalid project name. Use only alphanumeric characters and hyphens.');
        process.exit(1);
      }
      
      // Create project
      await createProject(name, options, logger);
    });
}
```

### **Example: Module Installation**
```typescript
// src/commands/install.ts
import { installModule } from '../utils/module-installer';
import { resolveDependencies } from '../utils/dependency-resolver';

export async function installCommand(moduleName: string, options: InstallOptions): Promise<void> {
  const logger = new Logger(options.verbose);
  
  try {
    // Resolve dependencies
    const dependencies = await resolveDependencies([moduleName]);
    logger.info(`Installing: ${dependencies.join(', ')}`);
    
    // Install each module
    for (const dep of dependencies) {
      await installModule(dep, options, logger);
    }
    
    logger.success('All modules installed successfully!');
  } catch (error) {
    logger.error(`Installation failed: ${error.message}`);
    process.exit(1);
  }
}
```

### **Example: Registry Integration**
```typescript
// src/utils/registry.ts
export class KimuRegistry {
  private static instance: KimuRegistry;
  private catalog: ModuleCatalog | null = null;
  
  static getInstance(): KimuRegistry {
    if (!KimuRegistry.instance) {
      KimuRegistry.instance = new KimuRegistry();
    }
    return KimuRegistry.instance;
  }
  
  async getModuleCatalog(): Promise<ModuleCatalog> {
    if (!this.catalog) {
      this.catalog = await this.fetchCatalog();
    }
    return this.catalog;
  }
  
  async getModuleInfo(moduleName: string): Promise<ModuleInfo | null> {
    const catalog = await this.getModuleCatalog();
    return catalog.modules[moduleName] || null;
  }
}
```

## FAQ & Troubleshooting
> **Agent Reminder:** Consult this section for common issues, edge cases, and best practices for error handling and CLI conventions.

**Q: How should commands handle network failures?**
A: Always implement retry logic with exponential backoff, cache responses when possible, and provide clear error messages with suggested solutions.

**Q: What's the standard pattern for user confirmation?**
A: Use inquirer.js for interactive prompts, always provide a `--yes` flag to skip confirmations in CI environments.

**Q: How should the CLI handle different project types?**
A: Detect project type using kimu.config.json, validate compatibility before operations, provide clear error messages for unsupported operations.

**Q: What's the convention for command output?**
A: Use chalk for colors, ora for spinners, consistent symbols (✅ ❌ ⚠️), and structured output that's both human and machine readable.

## Testing & Quality Assurance
> **Agent Reminder:** Follow these instructions to ensure high code quality, test coverage, and CLI reliability.

### **Testing Strategy**
- Unit tests for all utility functions using Jest
- Integration tests for complete command flows
- Mock external dependencies (file system, network)
- Test error conditions and edge cases
- Test CLI output and exit codes
- Use snapshot testing for help text and output formatting

### **Quality Standards**
- Maintain 90%+ test coverage
- Use ESLint with strict TypeScript rules
- Format code with Prettier
- Validate all user inputs
- Handle all error conditions gracefully
- Document all public APIs with TSDoc

### **Example Test Structure**
```typescript
// tests/commands/create.test.ts
describe('create command', () => {
  beforeEach(() => {
    mockFs.reset();
  });
  
  it('should create basic project structure', async () => {
    await createProject('test-app', { template: 'basic' });
    
    expect(mockFs.existsSync('test-app/package.json')).toBe(true);
    expect(mockFs.existsSync('test-app/src/main.ts')).toBe(true);
    expect(mockFs.existsSync('test-app/kimu.config.json')).toBe(true);
  });
  
  it('should handle invalid project names', async () => {
    await expect(createProject('invalid name!')).rejects.toThrow('Invalid project name');
  });
});
```

## Advanced Usage
> **Agent Reminder:** Review this section for advanced CLI patterns, plugin architecture, and extensibility features.

### **Plugin Architecture**
- Support for third-party plugins via npm packages
- Plugin discovery through naming convention (`kimu-plugin-*`)
- Plugin lifecycle hooks for command registration
- Secure plugin loading with validation

### **Configuration Management**
- Global configuration in user home directory
- Project-specific configuration in kimu.config.json
- Environment variable support for CI/CD
- Configuration validation and migration

### **Performance Optimization**
- Lazy loading of commands and dependencies
- Efficient caching strategies for registry data
- Parallel downloads for multiple modules
- Progress reporting for long operations

## Glossary
> **Agent Reminder:** Refer to the glossary for precise definitions of key concepts and terminology used throughout the CLI project.

- **CLI**: Command Line Interface - the kimu-cli tool itself
- **Command**: A specific operation like `create`, `install`, `list`
- **Module**: Reusable functionality packages from kimu-modules
- **Extension**: UI components and complete features
- **Template**: Project scaffolding templates for different app types
- **Registry**: The kimu-modules repository containing available modules
- **Catalog**: JSON file listing all available modules and extensions
- **Project**: A KIMU application created and managed by the CLI
- **Workspace**: Directory containing a KIMU project
- **Dependency Resolution**: Process of determining required modules for installation

## Integration with KIMU Ecosystem

### **Relationship to Other KIMU Projects**
- **kimu-core**: Uses as base framework for created projects
- **kimu-modules**: Primary source for installable modules
- **kimu-docs**: References for help text and documentation
- **Community projects**: Supports installation from various sources

### **KIMU-CLI Responsibilities vs KIMU-Core**

#### **KIMU-CLI Responsibilities** (Developer Tooling)
```bash
✅ Project creation and scaffolding
✅ Module and extension installation/management  
✅ Build orchestration and optimization
✅ Development server and hot reload
✅ Dependency resolution and compatibility checking
✅ Template management and code generation
✅ Project diagnostics and health checks
✅ Registry integration and package discovery
✅ Version management and updates
✅ Configuration management (kimu.config.json)
```

#### **KIMU-Core Responsibilities** (Runtime Framework)
```typescript
✅ Component lifecycle management (onInit, onRender, etc.)
✅ Web Components registration and rendering
✅ Extension loading and management at runtime
✅ Module service provision and dependency injection
✅ Event system and component communication
✅ DOM manipulation and reactive updates
✅ Application state management
✅ Asset loading and caching
✅ Error handling and boundaries
✅ Performance optimization (rendering, memory)
```

#### **Clear Separation of Concerns**
```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT TIME                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   KIMU-CLI                              │    │
│  │  • Project creation       • Module installation        │    │
│  │  • Build orchestration   • Dependency resolution       │    │
│  │  • Development tools     • Template management         │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      RUNTIME                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  KIMU-CORE                              │    │
│  │  • Component rendering   • Extension loading           │    │
│  │  • Lifecycle management  • Service provision           │    │
│  │  • Event handling       • State management             │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### **Version Compatibility**
- Maintain compatibility matrix with kimu-core versions
- Check version compatibility before module installation
- Provide clear error messages for incompatible versions
- Support multiple kimu-core versions when possible

#### **Compatibility Checking Implementation**
```typescript
// Example compatibility verification
interface VersionCompatibility {
  kimuCli: string;           // Current CLI version
  kimuCore: {
    required: string;        // Required by CLI
    installed?: string;      // In current project
    compatible: boolean;     // Compatibility status
  };
  modules: {
    [name: string]: {
      required: string;      // Required by module
      installed: string;     // Currently installed
      compatible: boolean;   // Compatibility status
    };
  };
}

async function checkProjectCompatibility(): Promise<VersionCompatibility> {
  const cliVersion = require('../package.json').version;
  const projectKimuCore = await getInstalledKimuCoreVersion();
  const requiredKimuCore = getRequiredKimuCoreVersion(cliVersion);
  
  return {
    kimuCli: cliVersion,
    kimuCore: {
      required: requiredKimuCore,
      installed: projectKimuCore,
      compatible: semver.satisfies(projectKimuCore, requiredKimuCore)
    },
    modules: await checkModulesCompatibility()
  };
}
```

### **Future Roadmap**
- Integration with kimu marketplace when available
- Support for custom registries and private modules
- Advanced project templates and generators
- Integration with popular development tools and IDEs
- Plugin ecosystem for extending CLI functionality

---

## Security Considerations

### **Safe File Operations**
- Always validate and sanitize file paths
- Use secure temporary directories for downloads
- Implement proper cleanup for failed operations
- Never execute downloaded code without explicit user consent

### **Network Security**
- Use HTTPS for all registry communications
- Validate downloaded content checksums when available
- Implement proper SSL certificate validation
- Support proxy configurations for corporate environments

### **Input Validation**
- Sanitize all user inputs and command arguments
- Validate project names and module names
- Check for path traversal attacks in file operations
- Implement rate limiting for registry requests

---

> **Agent Reminder:** Always follow the patterns and conventions outlined in this guide. When implementing new features, refer to existing commands for consistency. Update this documentation when adding new patterns or changing existing ones. The CLI should feel professional, reliable, and intuitive for all KIMU framework users.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UnicoVerso)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/UnicoVerso)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
