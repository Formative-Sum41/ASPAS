# Application Packaging Automation System (APAS) Data Models

## Core Application Entities / Domain Objects

### Package

- **Description:** Represents a complete application package, including all associated artifacts and metadata.
- **Schema / Interface Definition:**
  ```typescript
  export interface Package {
    id: string;                 // Unique identifier
    name: string;               // Package name
    version: string;            // Package version
    publisher: string;          // Application publisher
    description: string;        // Package description
    status: PackageStatus;      // Current status of the package
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
    installer_id: string;       // Reference to the installer
    psadt_script_id: string;    // Reference to the PSADT script
    wdac_policy_id: string;     // Reference to the WDAC policy
    documentation_id: string;   // Reference to the documentation
    metadata: Record<string, any>; // Additional metadata
    tags: string[];             // Classification tags
  }

  export enum PackageStatus {
    DRAFT = 'draft',
    ANALYZING = 'analyzing',
    GENERATING_SCRIPT = 'generating_script',
    CREATING_POLICY = 'creating_policy',
    TESTING = 'testing',
    WAITING_APPROVAL = 'waiting_approval',
    COMPLETED = 'completed',
    FAILED = 'failed'
  }
  ```
- **Validation Rules:**
  - `name` must be at least 3 characters
  - `version` must follow semantic versioning
  - `publisher` must be at least 2 characters
  - `status` must be a valid PackageStatus enum value

### Installer

- **Description:** Represents an application installer file with its metadata and analysis results.
- **Schema / Interface Definition:**
  ```typescript
  export interface Installer {
    id: string;                 // Unique identifier
    filename: string;           // Original filename
    file_path: string;          // Path to the stored file
    file_size: number;          // File size in bytes
    file_hash: string;          // File hash for integrity
    file_type: InstallerType;   // Type of installer
    framework: string;          // Installation framework
    architecture: Architecture; // Target architecture
    metadata: InstallerMetadata; // Extracted metadata
    silent_switches: string;    // Detected silent installation parameters
    analysis_results: AnalysisResults; // Results of the analysis
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
  }

  export enum InstallerType {
    MSI = 'msi',
    EXE = 'exe',
    MSIX = 'msix',
    APPX = 'appx',
    ZIP = 'zip',
    OTHER = 'other'
  }

  export enum Architecture {
    X86 = 'x86',
    X64 = 'x64',
    ARM = 'arm',
    ARM64 = 'arm64',
    UNIVERSAL = 'universal'
  }

  export interface InstallerMetadata {
    product_name: string;       // Product name
    product_version: string;    // Product version
    product_publisher: string;  // Product publisher
    product_language: string;   // Product language code
    product_code?: string;      // MSI product code
    upgrade_code?: string;      // MSI upgrade code
    requires_elevation: boolean; // Whether elevation is required
    additional_metadata: Record<string, any>; // Additional metadata
  }

  export interface AnalysisResults {
    confidence_score: number;   // Confidence in the analysis
    detected_requirements: string[]; // Detected requirements
    potential_issues: string[]; // Potential issues
    recommendations: string[];  // Recommendations
    external_research: ExternalResearch[]; // External research results
    analysis_logs: string[];    // Analysis logs
  }

  export interface ExternalResearch {
    source: string;             // Research source
    url: string;                // Source URL
    relevance_score: number;    // Relevance score
    information: string;        // Extracted information
  }
  ```
- **Validation Rules:**
  - `filename` must not be empty
  - `file_size` must be greater than 0
  - `file_hash` must be a valid hash string
  - `file_type` must be a valid InstallerType enum value
  - `architecture` must be a valid Architecture enum value
  - `confidence_score` must be between 0 and 1

### PSADTScript

- **Description:** Represents a PowerShell AppDeploy Toolkit script generated for an application.
- **Schema / Interface Definition:**
  ```typescript
  export interface PSADTScript {
    id: string;                 // Unique identifier
    package_id: string;         // Reference to the package
    template_id: string;        // Reference to the template used
    template_version: string;   // Template version
    script_content: string;     // Full script content
    pre_installation: string[]; // Pre-installation commands
    installation: string[];     // Installation commands
    post_installation: string[]; // Post-installation commands
    custom_functions: string[]; // Custom functions
    parameters: Record<string, any>; // Script parameters
    test_results: TestResult[]; // Results of script testing
    version_history: VersionEntry[]; // Version history
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
  }

  export interface TestResult {
    id: string;                 // Test identifier
    timestamp: Date;            // Test timestamp
    success: boolean;           // Whether the test succeeded
    logs: string[];             // Test logs
    execution_time: number;     // Execution time in seconds
    exit_code: number;          // Script exit code
    issues: string[];           // Identified issues
  }

  export interface VersionEntry {
    version: number;            // Version number
    timestamp: Date;            // Version timestamp
    changes: string[];          // Changes in this version
    created_by: string;         // Creator identifier
  }
  ```
- **Validation Rules:**
  - `script_content` must not be empty
  - `template_version` must follow semantic versioning
  - `installation` must contain at least one command
  - `test_results` may be empty during script generation

### WDACPolicy

- **Description:** Represents a Windows Defender Application Control policy for an application.
- **Schema / Interface Definition:**
  ```typescript
  export interface WDACPolicy {
    id: string;                 // Unique identifier
    package_id: string;         // Reference to the package
    policy_name: string;        // Policy name
    policy_type: PolicyType;    // Policy type
    policy_content: string;     // Full policy content (XML)
    policy_guid: string;        // Policy GUID
    policy_version: number;     // Policy version
    is_supplemental: boolean;   // Whether it's a supplemental policy
    base_policy_guid?: string;  // Base policy GUID for supplemental policies
    rules: PolicyRule[];        // Policy rules
    signing_info: SigningInfo;  // Information about code signing
    test_results: PolicyTestResult[]; // Results of policy testing
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
  }

  export enum PolicyType {
    BASE = 'base',
    SUPPLEMENTAL = 'supplemental',
    AUDIT = 'audit',
    ENFORCEMENT = 'enforcement'
  }

  export interface PolicyRule {
    id: string;                 // Rule identifier
    rule_type: RuleType;        // Rule type
    rule_level: RuleLevel;      // Rule level
    value: string;              // Rule value
    description: string;        // Rule description
    is_auto_generated: boolean; // Whether it was auto-generated
  }

  export enum RuleType {
    PUBLISHER = 'publisher',
    FILE_PATH = 'file_path',
    FILE_HASH = 'file_hash',
    FILE_NAME = 'file_name'
  }

  export enum RuleLevel {
    ALLOW = 'allow',
    DENY = 'deny',
    AUDIT = 'audit'
  }

  export interface SigningInfo {
    signed_files: number;       // Number of signed files
    unsigned_files: number;     // Number of unsigned files
    publishers: Publisher[];    // Unique publishers
  }

  export interface Publisher {
    name: string;               // Publisher name
    thumbprint: string;         // Certificate thumbprint
    file_count: number;         // Number of files from this publisher
  }

  export interface PolicyTestResult {
    id: string;                 // Test identifier
    timestamp: Date;            // Test timestamp
    mode: PolicyType;           // Test mode
    success: boolean;           // Whether the test succeeded
    violations: Violation[];    // Identified violations
    logs: string[];             // Test logs
  }

  export interface Violation {
    file_path: string;          // Path to the violating file
    reason: string;             // Violation reason
    recommended_rule: PolicyRule; // Recommended rule to fix the violation
  }
  ```
- **Validation Rules:**
  - `policy_name` must not be empty
  - `policy_guid` must be a valid GUID
  - `policy_content` must be valid XML
  - `policy_type` must be a valid PolicyType enum value
  - `is_supplemental` must be true if `base_policy_guid` is provided

### Documentation

- **Description:** Represents the documentation generated for a package.
- **Schema / Interface Definition:**
  ```typescript
  export interface Documentation {
    id: string;                 // Unique identifier
    package_id: string;         // Reference to the package
    content_markdown: string;   // Full documentation content in Markdown
    sections: DocumentSection[]; // Documentation sections
    images: DocumentImage[];    // Documentation images
    metadata: Record<string, any>; // Additional metadata
    version: number;            // Documentation version
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
  }

  export interface DocumentSection {
    id: string;                 // Section identifier
    title: string;              // Section title
    content: string;            // Section content
    order: number;              // Section order
    parent_id?: string;         // Parent section identifier
  }

  export interface DocumentImage {
    id: string;                 // Image identifier
    filename: string;           // Image filename
    file_path: string;          // Path to the stored image
    caption: string;            // Image caption
    alt_text: string;           // Alternative text
    width: number;              // Image width
    height: number;             // Image height
  }
  ```
- **Validation Rules:**
  - `content_markdown` must not be empty
  - `version` must be a positive integer
  - `sections` must have at least one item
  - Section `order` values must be unique within the same parent

### User

- **Description:** Represents a system user who can interact with the APAS system.
- **Schema / Interface Definition:**
  ```typescript
  export interface User {
    id: string;                 // Unique identifier
    username: string;           // Username
    email: string;              // Email address
    full_name: string;          // Full name
    role: UserRole;             // User role
    preferences: UserPreferences; // User preferences
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
    last_login_at?: Date;       // Last login timestamp
  }

  export enum UserRole {
    ADMIN = 'admin',
    PACKAGER = 'packager',
    APPROVER = 'approver',
    VIEWER = 'viewer'
  }

  export interface UserPreferences {
    theme: 'light' | 'dark' | 'system'; // UI theme preference
    dashboard_layout: any;      // Dashboard layout configuration
    notification_settings: NotificationSettings; // Notification settings
  }

  export interface NotificationSettings {
    email_notifications: boolean; // Whether to send email notifications
    desktop_notifications: boolean; // Whether to show desktop notifications
    notification_events: string[]; // Events to be notified about
  }
  ```
- **Validation Rules:**
  - `username` must be at least 3 characters and unique
  - `email` must be a valid email address and unique
  - `full_name` must not be empty
  - `role` must be a valid UserRole enum value

### Event

- **Description:** Represents an event in the system for inter-agent communication and audit purposes.
- **Schema / Interface Definition:**
  ```typescript
  export interface Event {
    id: string;                 // Unique identifier
    event_type: string;         // Event type
    topic: string;              // Event topic
    payload: Record<string, any>; // Event payload
    source: string;             // Event source
    correlation_id: string;     // Correlation identifier
    package_id?: string;        // Reference to the package
    timestamp: Date;            // Event timestamp
    processed: boolean;         // Whether the event has been processed
    processing_time?: number;   // Processing time in milliseconds
  }
  ```
- **Validation Rules:**
  - `event_type` must not be empty
  - `topic` must follow the topic naming convention
  - `source` must identify a valid system component
  - `correlation_id` must not be empty

### KnowledgeItem

- **Description:** Represents a knowledge item in the system's knowledge base.
- **Schema / Interface Definition:**
  ```typescript
  export interface KnowledgeItem {
    id: string;                 // Unique identifier
    title: string;              // Item title
    content: string;            // Item content
    content_type: ContentType;  // Content type
    category: string;           // Item category
    tags: string[];             // Classification tags
    source_package_id?: string; // Source package identifier
    source_url?: string;        // Source URL
    embedding: number[];        // Vector embedding
    metadata: Record<string, any>; // Additional metadata
    relevance_score: number;    // Relevance score
    created_at: Date;           // Creation timestamp
    updated_at: Date;           // Last update timestamp
  }

  export enum ContentType {
    INSTALLER_PATTERN = 'installer_pattern',
    PSADT_SCRIPT = 'psadt_script',
    WDAC_POLICY = 'wdac_policy',
    TROUBLESHOOTING = 'troubleshooting',
    BEST_PRACTICE = 'best_practice',
    GENERAL = 'general'
  }
  ```
- **Validation Rules:**
  - `title` must not be empty
  - `content` must not be empty
  - `content_type` must be a valid ContentType enum value
  - `category` must not be empty
  - `embedding` must be a vector of appropriate dimensionality

## Database Schema

### packages Table

```sql
CREATE TABLE packages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    version VARCHAR(50) NOT NULL,
    publisher VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    installer_id UUID REFERENCES installers(id),
    psadt_script_id UUID REFERENCES psadt_scripts(id),
    wdac_policy_id UUID REFERENCES wdac_policies(id),
    documentation_id UUID REFERENCES documentation(id),
    metadata JSONB DEFAULT '{}'::jsonb,
    tags TEXT[] DEFAULT '{}'::text[],
    
    CONSTRAINT valid_status CHECK (status IN ('draft', 'analyzing', 'generating_script', 'creating_policy', 'testing', 'waiting_approval', 'completed', 'failed'))
);

CREATE INDEX idx_packages_status ON packages(status);
CREATE INDEX idx_packages_tags ON packages USING GIN(tags);
```

### installers Table

```sql
CREATE TABLE installers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    filename VARCHAR(255) NOT NULL,
    file_path VARCHAR(255) NOT NULL,
    file_size BIGINT NOT NULL,
    file_hash VARCHAR(128) NOT NULL,
    file_type VARCHAR(20) NOT NULL,
    framework VARCHAR(50),
    architecture VARCHAR(20) NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    silent_switches TEXT,
    analysis_results JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT valid_file_type CHECK (file_type IN ('msi', 'exe', 'msix', 'appx', 'zip', 'other')),
    CONSTRAINT valid_architecture CHECK (architecture IN ('x86', 'x64', 'arm', 'arm64', 'universal'))
);

CREATE INDEX idx_installers_file_type ON installers(file_type);
CREATE INDEX idx_installers_architecture ON installers(architecture);
```

### psadt_scripts Table

```sql
CREATE TABLE psadt_scripts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    package_id UUID REFERENCES packages(id),
    template_id UUID REFERENCES templates(id),
    template_version VARCHAR(50) NOT NULL,
    script_content TEXT NOT NULL,
    pre_installation JSONB DEFAULT '[]'::jsonb,
    installation JSONB NOT NULL,
    post_installation JSONB DEFAULT '[]'::jsonb,
    custom_functions JSONB DEFAULT '[]'::jsonb,
    parameters JSONB DEFAULT '{}'::jsonb,
    test_results JSONB DEFAULT '[]'::jsonb,
    version_history JSONB DEFAULT '[]'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### wdac_policies Table

```sql
CREATE TABLE wdac_policies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    package_id UUID REFERENCES packages(id),
    policy_name VARCHAR(255) NOT NULL,
    policy_type VARCHAR(20) NOT NULL,
    policy_content TEXT NOT NULL,
    policy_guid VARCHAR(36) NOT NULL,
    policy_version INTEGER NOT NULL,
    is_supplemental BOOLEAN NOT NULL DEFAULT FALSE,
    base_policy_guid VARCHAR(36),
    rules JSONB NOT NULL DEFAULT '[]'::jsonb,
    signing_info JSONB NOT NULL DEFAULT '{}'::jsonb,
    test_results JSONB DEFAULT '[]'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT valid_policy_type CHECK (policy_type IN ('base', 'supplemental', 'audit', 'enforcement')),
    CONSTRAINT supplemental_requires_base CHECK (NOT is_supplemental OR base_policy_guid IS NOT NULL)
);

CREATE INDEX idx_wdac_policies_policy_type ON wdac_policies(policy_type);
```

### documentation Table

```sql
CREATE TABLE documentation (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    package_id UUID REFERENCES packages(id),
    content_markdown TEXT NOT NULL,
    sections JSONB NOT NULL,
    images JSONB DEFAULT '[]'::jsonb,
    metadata JSONB DEFAULT '{}'::jsonb,
    version INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### users Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,
    preferences JSONB DEFAULT '{
        "theme": "dark",
        "dashboard_layout": {},
        "notification_settings": {
            "email_notifications": true,
            "desktop_notifications": true,
            "notification_events": []
        }
    }'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login_at TIMESTAMP WITH TIME ZONE,
    
    CONSTRAINT valid_role CHECK (role IN ('admin', 'packager', 'approver', 'viewer'))
);

CREATE INDEX idx_users_role ON users(role);
```

### events Table

```sql
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type VARCHAR(100) NOT NULL,
    topic VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    source VARCHAR(100) NOT NULL,
    correlation_id VARCHAR(100) NOT NULL,
    package_id UUID REFERENCES packages(id),
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    processed BOOLEAN NOT NULL DEFAULT FALSE,
    processing_time INTEGER
);

CREATE INDEX idx_events_event_type ON events(event_type);
CREATE INDEX idx_events_topic ON events(topic);
CREATE INDEX idx_events_correlation_id ON events(correlation_id);
CREATE INDEX idx_events_processed ON events(processed);
```

### knowledge_items Table

```sql
CREATE TABLE knowledge_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    content_type VARCHAR(50) NOT NULL,
    category VARCHAR(100) NOT NULL,
    tags TEXT[] DEFAULT '{}'::text[],
    source_package_id UUID REFERENCES packages(id),
    source_url TEXT,
    embedding vector(384), -- Using pgvector extension
    metadata JSONB DEFAULT '{}'::jsonb,
    relevance_score FLOAT DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT valid_content_type CHECK (content_type IN ('installer_pattern', 'psadt_script', 'wdac_policy', 'troubleshooting', 'best_practice', 'general'))
);

CREATE INDEX idx_knowledge_items_content_type ON knowledge_items(content_type);
CREATE INDEX idx_knowledge_items_category ON knowledge_items(category);
CREATE INDEX idx_knowledge_items_tags ON knowledge_items USING GIN(tags);
CREATE INDEX idx_knowledge_items_embedding ON knowledge_items USING ivfflat (embedding vector_cosine_ops);
```

## Supabase Integration

The database will be implemented using Supabase, which provides:

1. PostgreSQL database with pgvector extension for vector similarity search
2. Storage for installer files and documentation images
3. Real-time subscriptions for UI updates
4. Authentication and authorization

### Key Supabase Features Used

- **PostgreSQL**: Core database functionality
- **pgvector**: Vector embeddings for the knowledge repository
- **Storage Buckets**: For installer files, scripts, documentation, and images
- **Row-Level Security (RLS)**: For fine-grained access control
- **Real-time Subscriptions**: For event-driven UI updates
- **Functions**: For complex database operations

### Example Supabase JavaScript Client Usage

```typescript
// Initialize Supabase client
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
);

// Query packages
const { data: packages, error } = await supabase
  .from('packages')
  .select('*')
  .eq('status', 'completed');

// Insert a new package
const { data: newPackage, error } = await supabase
  .from('packages')
  .insert({
    name: 'Example Application',
    version: '1.0.0',
    publisher: 'Example Publisher',
    description: 'Example description',
    status: 'draft'
  })
  .single();

// Upload an installer file
const { data: fileData, error } = await supabase.storage
  .from('installers')
  .upload(`${uuid()}/installer.exe`, fileBuffer);
```

### Example Supabase Python Client Usage

```python
from supabase import create_client
import os

url = os.environ.get("SUPABASE_URL")
key = os.environ.get("SUPABASE_KEY")
supabase = create_client(url, key)

# Query packages
packages = supabase.table("packages").select("*").eq("status", "completed").execute()

# Insert a new package
new_package = supabase.table("packages").insert({
    "name": "Example Application",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "description": "Example description",
    "status": "draft"
}).execute()

# Upload an installer file
file_upload = supabase.storage.from_("installers").upload(
    f"{uuid}/installer.exe",
    file_data
)
```

## State File Schemas

### API Response Schemas

#### Package Response

```json
{
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique identifier for the package"
    },
    "name": {
      "type": "string",
      "description": "Package name"
    },
    "version": {
      "type": "string",
      "description": "Package version"
    },
    "publisher": {
      "type": "string",
      "description": "Application publisher"
    },
    "description": {
      "type": "string",
      "description": "Package description"
    },
    "status": {
      "type": "string",
      "enum": ["draft", "analyzing", "generating_script", "creating_policy", "testing", "waiting_approval", "completed", "failed"],
      "description": "Current status of the package"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "Creation timestamp"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "Last update timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Additional metadata"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Classification tags"
    }
  },
  "required": ["id", "name", "version", "publisher", "status", "created_at", "updated_at"]
}
```

#### Installer Analysis Request

```json
{
  "type": "object",
  "properties": {
    "file_id": {
      "type": "string",
      "format": "uuid",
      "description": "ID of the uploaded file"
    },
    "filename": {
      "type": "string",
      "description": "Original filename"
    },
    "file_path": {
      "type": "string",
      "description": "Path to the stored file"
    },
    "options": {
      "type": "object",
      "properties": {
        "detailed_analysis": {
          "type": "boolean",
          "default": true,
          "description": "Whether to perform detailed analysis"
        },
        "external_research": {
          "type": "boolean",
          "default": true,
          "description": "Whether to perform external research"
        },
        "ai_assistance": {
          "type": "boolean",
          "default": true,
          "description": "Whether to use AI assistance"
        }
      }
    }
  },
  "required": ["file_id", "filename", "file_path"]
}
```

## Change Log

| Change        | Date       | Version | Description                 | Author         |
| ------------- | ---------- | ------- | --------------------------- | -------------- |
| Initial draft | 2025-05-08 | 0.1     | Initial data models definition | Architect Agent |
