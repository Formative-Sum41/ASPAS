# Application Packaging Automation System (APAS) Environment Variables

## Configuration Loading Mechanism

APAS uses a layered approach to environment variable management, prioritizing different sources in the following order:

1. **Operating System Environment Variables**: Highest priority, can override all other sources
2. **`.env` File**: Located in the project root, loaded via `python-dotenv` in backend and `dotenv` in frontend
3. **Default Values**: Hardcoded defaults as a fallback when variables are not defined elsewhere

### Backend Configuration Loading

The backend uses Python's `pydantic-settings` for type-safe configuration management. The configuration is defined in `backend/apas/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, validator
import os
from typing import Optional, List, Dict, Any

class Settings(BaseSettings):
    # Application settings
    APP_NAME: str = "APAS"
    APP_VERSION: str = "0.1.0"
    APP_ENV: str = "development"  # development, testing, production
    DEBUG: bool = True
    API_PREFIX: str = "/api/v1"
    SECRET_KEY: str
    
    # CORS settings
    CORS_ORIGINS: List[str] = ["http://localhost:3000"]
    CORS_CREDENTIALS: bool = True
    
    # Database settings
    SUPABASE_URL: str
    SUPABASE_KEY: str
    SUPABASE_SERVICE_KEY: Optional[str] = None
    
    # AI model settings
    MODEL_PATH: Optional[str] = None
    USE_CLOUD_MODELS: bool = False
    OPENAI_API_KEY: Optional[str] = None
    MODEL_MEMORY_LIMIT: int = 8  # GB
    
    # Logging settings
    LOG_LEVEL: str = "INFO"
    LOG_FORMAT: str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    
    # File paths
    TEMP_DIR: str = "/tmp/apas"
    DATA_DIR: str = "./data"
    TEMPLATES_DIR: str = "./data/templates"
    
    # Agent settings
    AGENT_TIMEOUT: int = 300  # seconds
    AGENT_MAX_RETRIES: int = 3
    
    # PowerShell settings
    POWERSHELL_PATH: str = "powershell.exe"
    PSADT_VERSION: str = "3.9.3"
    
    @validator("APP_ENV")
    def validate_app_env(cls, v):
        if v not in ["development", "testing", "production"]:
            raise ValueError("APP_ENV must be one of: development, testing, production")
        return v
    
    @validator("LOG_LEVEL")
    def validate_log_level(cls, v):
        if v not in ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]:
            raise ValueError("LOG_LEVEL must be one of: DEBUG, INFO, WARNING, ERROR, CRITICAL")
        return v
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

### Frontend Configuration Loading

The frontend uses a `.env` file with environment variables prefixed with `NEXT_PUBLIC_` or `VITE_` (depending on the bundler):

```typescript
// frontend/src/config.ts

export interface Config {
  apiUrl: string;
  supabaseUrl: string;
  supabaseAnonKey: string;
  environment: 'development' | 'testing' | 'production';
  enableDevTools: boolean;
  logLevel: 'debug' | 'info' | 'warn' | 'error';
  maxFileSize: number;
}

export const config: Config = {
  apiUrl: import.meta.env.VITE_API_URL || 'http://localhost:8000/api/v1',
  supabaseUrl: import.meta.env.VITE_SUPABASE_URL || '',
  supabaseAnonKey: import.meta.env.VITE_SUPABASE_ANON_KEY || '',
  environment: (import.meta.env.VITE_APP_ENV || 'development') as 'development' | 'testing' | 'production',
  enableDevTools: import.meta.env.VITE_ENABLE_DEV_TOOLS === 'true',
  logLevel: (import.meta.env.VITE_LOG_LEVEL || 'info') as 'debug' | 'info' | 'warn' | 'error',
  maxFileSize: parseInt(import.meta.env.VITE_MAX_FILE_SIZE || '5242880', 10), // 5MB default
};
```

## Required Variables

### Backend Environment Variables

| Variable Name        | Description                                     | Example / Default Value         | Required? | Sensitive? |
| :------------------- | :---------------------------------------------- | :------------------------------ | :-------- | :--------- |
| `APP_ENV`            | Application environment                         | `development`                   | No        | No         |
| `DEBUG`              | Enable debug mode                               | `True`                          | No        | No         |
| `SECRET_KEY`         | Secret key for encryption                       | `your-secret-key`               | Yes       | Yes        |
| `SUPABASE_URL`       | Supabase URL                                    | `https://xxx.supabase.co`       | Yes       | No         |
| `SUPABASE_KEY`       | Supabase anonymous API key                      | `eyJhbGciOiJIUzI1NiIsInR5...`   | Yes       | Yes        |
| `SUPABASE_SERVICE_KEY` | Supabase service key for admin operations     | `eyJhbGciOiJIUzI1NiIsInR5...`   | No        | Yes        |
| `MODEL_PATH`         | Path to local AI models                         | `/path/to/models`               | No        | No         |
| `USE_CLOUD_MODELS`   | Whether to use cloud-based AI models            | `False`                         | No        | No         |
| `OPENAI_API_KEY`     | OpenAI API key (if using cloud models)          | `sk-...`                        | No        | Yes        |
| `MODEL_MEMORY_LIMIT` | Memory limit for AI models in GB                | `8`                             | No        | No         |
| `LOG_LEVEL`          | Logging level                                   | `INFO`                          | No        | No         |
| `TEMP_DIR`           | Temporary directory for file processing         | `/tmp/apas`                     | No        | No         |
| `DATA_DIR`           | Data directory for persistent storage           | `./data`                        | No        | No         |
| `TEMPLATES_DIR`      | Directory containing PSADT templates            | `./data/templates`              | No        | No         |
| `AGENT_TIMEOUT`      | Timeout for agent operations in seconds         | `300`                           | No        | No         |
| `AGENT_MAX_RETRIES`  | Maximum number of retries for agent operations  | `3`                             | No        | No         |
| `POWERSHELL_PATH`    | Path to PowerShell executable                   | `powershell.exe`                | No        | No         |
| `PSADT_VERSION`      | Version of PSADT to use                         | `3.9.3`                         | No        | No         |

### Frontend Environment Variables

| Variable Name               | Description                                     | Example / Default Value         | Required? | Sensitive? |
| :-------------------------- | :---------------------------------------------- | :------------------------------ | :-------- | :--------- |
| `VITE_API_URL`              | Backend API URL                                 | `http://localhost:8000/api/v1`  | Yes       | No         |
| `VITE_SUPABASE_URL`         | Supabase URL                                    | `https://xxx.supabase.co`       | Yes       | No         |
| `VITE_SUPABASE_ANON_KEY`    | Supabase anonymous API key                      | `eyJhbGciOiJIUzI1NiIsInR5...`   | Yes       | Yes        |
| `VITE_APP_ENV`              | Application environment                         | `development`                   | No        | No         |
| `VITE_ENABLE_DEV_TOOLS`     | Enable development tools                        | `true`                          | No        | No         |
| `VITE_LOG_LEVEL`            | Logging level                                   | `info`                          | No        | No         |
| `VITE_MAX_FILE_SIZE`        | Maximum file size for uploads in bytes          | `5242880`                       | No        | No         |

## Environment-Specific Configurations

### Development Environment

For local development, create a `.env` file in the project root with the following variables:

```bash
# Application settings
APP_ENV=development
DEBUG=True
SECRET_KEY=dev-secret-key-change-me-in-production

# Supabase settings
SUPABASE_URL=http://localhost:54321
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5...
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5...

# AI model settings
MODEL_PATH=./models
USE_CLOUD_MODELS=False
MODEL_MEMORY_LIMIT=8

# Logging settings
LOG_LEVEL=DEBUG

# Frontend settings
VITE_API_URL=http://localhost:8000/api/v1
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5...
VITE_APP_ENV=development
VITE_ENABLE_DEV_TOOLS=true
VITE_LOG_LEVEL=debug
```

### Testing Environment

For testing, create a `.env.test` file:

```bash
# Application settings
APP_ENV=testing
DEBUG=False
SECRET_KEY=test-secret-key

# Supabase settings
SUPABASE_URL=http://localhost:54321
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5...
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5...

# AI model settings
MODEL_PATH=./models
USE_CLOUD_MODELS=False
MODEL_MEMORY_LIMIT=4

# Logging settings
LOG_LEVEL=INFO

# Frontend settings
VITE_API_URL=http://localhost:8000/api/v1
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5...
VITE_APP_ENV=testing
VITE_ENABLE_DEV_TOOLS=false
VITE_LOG_LEVEL=info
```

### Production Environment

For production, set environment variables directly on the system or use a secure environment variable management system:

```bash
# Application settings
APP_ENV=production
DEBUG=False
SECRET_KEY=<strong-random-secret-key>

# Supabase settings
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=<your-supabase-anon-key>
SUPABASE_SERVICE_KEY=<your-supabase-service-key>

# AI model settings
MODEL_PATH=/opt/apas/models
USE_CLOUD_MODELS=True
OPENAI_API_KEY=<your-openai-api-key>
MODEL_MEMORY_LIMIT=16

# Logging settings
LOG_LEVEL=INFO

# Frontend settings
VITE_API_URL=https://your-api-url.com/api/v1
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=<your-supabase-anon-key>
VITE_APP_ENV=production
VITE_ENABLE_DEV_TOOLS=false
VITE_LOG_LEVEL=warn
```

## Notes

### Secrets Management

- **Development**: Use `.env` files for local development (gitignored).
- **Testing**: Use `.env.test` files for testing environments.
- **Production**: Use environment variables set at the system level or through a secrets management system.
- **CI/CD**: Use secure environment variables in your CI/CD pipeline.

### Example .env File

A `.env.example` file is maintained in the repository with placeholder values for developers:

```bash
# Application settings
APP_ENV=development
DEBUG=True
SECRET_KEY=dev-secret-key-change-me

# Supabase settings
SUPABASE_URL=http://localhost:54321
SUPABASE_KEY=your-supabase-anon-key
SUPABASE_SERVICE_KEY=your-supabase-service-key

# AI model settings
MODEL_PATH=./models
USE_CLOUD_MODELS=False
OPENAI_API_KEY=your-openai-api-key
MODEL_MEMORY_LIMIT=8

# Logging settings
LOG_LEVEL=DEBUG

# PowerShell settings
POWERSHELL_PATH=powershell.exe
PSADT_VERSION=3.9.3

# Frontend settings
VITE_API_URL=http://localhost:8000/api/v1
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=your-supabase-anon-key
VITE_APP_ENV=development
VITE_ENABLE_DEV_TOOLS=true
VITE_LOG_LEVEL=debug
VITE_MAX_FILE_SIZE=5242880
```

### Validation

Environment variables are validated during application startup using Pydantic's validation capabilities. If required variables are missing or have invalid values, the application will fail to start with a clear error message.

## Change Log

| Change        | Date       | Version | Description                                | Author         |
| ------------- | ---------- | ------- | ------------------------------------------ | -------------- |
| Initial draft | 2025-05-08 | 0.1     | Initial environment variables specification | Architect Agent |
