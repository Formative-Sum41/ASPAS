# External Service Account Setup Guide

## Overview

This guide provides step-by-step instructions for setting up the external service accounts required by the Application Packaging Automation System (APAS). Following these procedures will ensure you have all necessary credentials and configurations to properly deploy and run the system.

## Required External Services

APAS requires the following external services:

1. **Supabase** - For database, authentication, storage, and vector operations
2. **Optional: OpenAI API** - For cloud model fallback when needed

## 1. Supabase Account Setup

Supabase provides the database, authentication, and storage capabilities for APAS. Follow these steps to set up your Supabase account and project.

### 1.1 Create Supabase Account

1. Visit [https://supabase.com/](https://supabase.com/)
2. Click on "Start your project" or "Sign Up"
3. Sign up using your email, GitHub, or Google account
4. Verify your email if required

### 1.2 Create a New Supabase Project

1. From the Supabase dashboard, click "New Project"
2. Enter a project name (e.g., "APAS")
3. Set a secure database password (keep this saved securely)
4. Choose a region closest to your deployment location
5. Choose the free tier for development or an appropriate paid tier for production
6. Click "Create new project"
7. Wait for the project to be provisioned (this may take a few minutes)

### 1.3 Get Project Credentials

1. Once your project is created, go to the project dashboard
2. Navigate to "Project Settings" > "API" in the sidebar
3. Note down the following credentials:
   - **Project URL**: Located in the "Project Configuration" section (e.g., `https://abcdefghijklm.supabase.co`)
   - **Project API Key (anon key)**: Located in the "Project API Keys" section 
   - **Project API Key (service_role key)**: Located in the "Project API Keys" section (keep this key secure)

### 1.4 Set Up Local Development Environment (Optional)

For local development, you can run Supabase locally:

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. Install the [Supabase CLI](https://github.com/supabase/cli)
3. Initialize Supabase in your project:
   ```bash
   npx supabase init
   ```
4. Start the local Supabase instance:
   ```bash
   npx supabase start
   ```
5. The CLI will output your local Supabase URL and keys, which typically are:
   - URL: `http://localhost:54321`
   - Anon Key: A long string starting with `eyJhbGciOiJIUzI1NiIsInR5...`
   - Service Role Key: A long string starting with `eyJhbGciOiJIUzI1NiIsInR5...`

### 1.5 Configure Database Schema

1. Navigate to the "SQL Editor" in your Supabase project dashboard
2. Run the database migration scripts located in `/supabase/migrations` in your APAS project
3. Alternatively, you can use the Supabase CLI to push migrations:
   ```bash
   npx supabase db push
   ```

### 1.6 Set Up Storage Buckets

APAS requires several storage buckets for different types of files:

1. Navigate to "Storage" in your Supabase project dashboard
2. Create the following buckets:
   - `installers` - For application installer files
   - `scripts` - For generated PSADT scripts
   - `policies` - For WDAC policies
   - `documentation` - For generated documentation
   - `templates` - For PSADT templates

For each bucket:
1. Click "Create bucket"
2. Enter the bucket name
3. Choose appropriate access permissions:
   - For development: "Public" bucket type
   - For production: "Private" bucket type
4. Click "Create bucket"

### 1.7 Set Up RLS Policies

For production environments, set up Row Level Security (RLS) policies:

1. Navigate to "Authentication" > "Policies" in your Supabase project dashboard
2. For each table, create appropriate RLS policies
3. At minimum, restrict access to authenticated users

## 2. OpenAI API Setup (Optional)

If you plan to use cloud model fallback for complex reasoning tasks, you'll need an OpenAI API key.

### 2.1 Create OpenAI Account

1. Visit [https://platform.openai.com/signup](https://platform.openai.com/signup)
2. Sign up for an account using your email or Google/Microsoft account
3. Complete the verification process

### 2.2 Set Up Billing

1. Navigate to "Billing" in your OpenAI dashboard
2. Add a payment method
3. Set usage limits to control costs

### 2.3 Create API Key

1. Navigate to "API keys" in your OpenAI dashboard
2. Click "Create new secret key"
3. Give your key a name (e.g., "APAS Production")
4. Copy the displayed API key and store it securely (you won't be able to view it again)

## 3. Configuring APAS with External Service Credentials

Once you have set up all required external services, you need to configure APAS to use them.

### 3.1 Backend Environment Variables

Add the following environment variables to your `.env` file or deployment environment:

```bash
# Supabase settings
SUPABASE_URL=your-project-url
SUPABASE_KEY=your-anon-key
SUPABASE_SERVICE_KEY=your-service-role-key

# OpenAI settings (optional)
USE_CLOUD_MODELS=True  # Set to True to enable cloud model fallback
OPENAI_API_KEY=your-openai-api-key
```

### 3.2 Frontend Environment Variables

Add the following environment variables for the frontend:

```bash
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-anon-key
```

## 4. Verifying External Service Configuration

After setting up all external services and configuring APAS, verify the configuration is working correctly:

### 4.1 Test Supabase Connection

1. Start the APAS application
2. Navigate to the Admin section
3. Run the connection test utility
4. Verify that all Supabase services (database, auth, storage) are working correctly

### 4.2 Test OpenAI Integration (if configured)

1. Navigate to the Admin section
2. Run the AI model test utility
3. Verify that cloud model fallback is working correctly

## 5. Troubleshooting Common Issues

### 5.1 Supabase Connection Issues

- **Issue**: Unable to connect to Supabase
  - **Solution**: Verify your `SUPABASE_URL` and `SUPABASE_KEY` are correct
  - **Solution**: Check if your IP is allowed in the Supabase project settings

- **Issue**: Permission denied errors
  - **Solution**: Check your RLS policies
  - **Solution**: Verify you're using the correct key (anon vs service_role) for the operation

- **Issue**: Migration errors
  - **Solution**: Check the SQL Editor for error details
  - **Solution**: Verify your Supabase version matches the expected version

### 5.2 OpenAI API Issues

- **Issue**: API key errors
  - **Solution**: Verify your `OPENAI_API_KEY` is correct
  - **Solution**: Check if your billing is set up correctly

- **Issue**: Rate limit errors
  - **Solution**: Implement rate limiting in your application
  - **Solution**: Increase your usage limits in the OpenAI dashboard

## 6. Security Considerations

### 6.1 Securing API Keys

- Never commit API keys to version control
- Use environment variables or secure key management systems
- Rotate keys periodically, especially for production environments

### 6.2 Access Control

- Implement proper access control for your Supabase project
- Use RLS policies to restrict data access
- Follow the principle of least privilege for service accounts

## 7. Maintenance Tasks

### 7.1 Regular Backups

- Set up regular database backups
- For production environments, enable point-in-time recovery in Supabase

### 7.2 Monitoring Usage

- Monitor your Supabase usage to avoid unexpected charges
- Set up spending limits for OpenAI API to control costs

## Reference Documentation

- [Supabase Documentation](https://supabase.com/docs)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- APAS Environment Variables (`docs/architecture/environment-vars.md`)
- APAS Database Schema (`docs/architecture/data-models.md`)
