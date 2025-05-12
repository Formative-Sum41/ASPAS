# Application Packaging Automation System (APAS) Testing Strategy

## Overall Philosophy & Goals

APAS follows a comprehensive testing approach guided by the "Testing Pyramid" principle, with more unit tests than integration tests, and more integration tests than end-to-end tests. Our testing strategy is designed to ensure high quality, reliability, and maintainability of the system while enabling confident refactoring and feature development.

- **Goal 1:** Achieve 80%+ code coverage for critical components including all AI agents and core services
- **Goal 2:** Ensure reliable packaging results with consistent outcomes for the same inputs
- **Goal 3:** Validate AI agent decision quality against known-good examples
- **Goal 4:** Verify system resilience with graceful handling of edge cases and failures
- **Goal 5:** Enable safe refactoring through comprehensive test coverage
- **Goal 6:** Validate explainable AI visualizations accurately represent agent reasoning

## Testing Levels

### Unit Tests

- **Scope:** Test individual functions, methods, or components in isolation. Focus on business logic, calculations, and conditional paths within a single module.
- **Tools:** Pytest for Python backend, Vitest for TypeScript/React frontend
- **Mocking/Stubbing:** 
  - Python: `pytest-mock`, `unittest.mock`
  - TypeScript: `vitest` mocking, `msw` for API mocking
  - AI Models: Mock models with predetermined responses
- **Location:** 
  - Backend: `backend/tests/unit/`
  - Frontend: `frontend/src/__tests__/unit/`
- **Expectations:** 
  - Should cover all significant logic paths
  - Fast execution (aim for <2 seconds per test)
  - Independent of external resources
  - High coverage of core components (>80%)

**Example Backend Unit Test:**
```python
def test_installer_analysis_agent_identifies_msi():
    # Given an MSI installer file mock
    installer_mock = MagicMock()
    installer_mock.filename = "test.msi"
    installer_mock.file_path = "/path/to/test.msi"
    
    # And a model mock with predetermined response
    model_mock = MagicMock()
    model_mock.predict.return_value = {
        "installer_type": "MSI",
        "confidence": 0.95
    }
    
    # When the installer analysis agent processes it
    agent = InstallerAnalysisAgent(model=model_mock)
    result = agent.analyze(installer_mock)
    
    # Then it should correctly identify the type
    assert result.installer_type == "MSI"
    assert result.confidence_score >= 0.9
    assert model_mock.predict.called
```

**Example Frontend Unit Test:**
```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import ConfidenceIndicator from '../../components/visualization/ConfidenceIndicator';

describe('ConfidenceIndicator', () => {
  it('displays high confidence correctly', () => {
    render(<ConfidenceIndicator value={0.95} />);
    const indicator = screen.getByTestId('confidence-indicator');
    expect(indicator).toHaveClass('bg-green-500');
    expect(screen.getByText('95%')).toBeInTheDocument();
  });

  it('displays low confidence correctly', () => {
    render(<ConfidenceIndicator value={0.3} />);
    const indicator = screen.getByTestId('confidence-indicator');
    expect(indicator).toHaveClass('bg-red-500');
    expect(screen.getByText('30%')).toBeInTheDocument();
  });
});
```

### Integration Tests

- **Scope:** Verify the interaction and collaboration between multiple components or modules. Test the flow of data and control within a specific feature or workflow slice.
- **Tools:** Pytest for backend, Vitest with React Testing Library for frontend
- **Strategy:** 
  - Test component interactions with real implementations when practical
  - Use in-memory database or local Supabase for data persistence tests
  - Test event flow through the event bus
  - Verify agent collaboration through orchestration core
- **Location:** 
  - Backend: `backend/tests/integration/`
  - Frontend: `frontend/src/__tests__/integration/`
- **Expectations:** 
  - Focus on component boundaries and contracts
  - Medium execution speed (aim for <10 seconds per test)
  - Cover critical interaction paths
  - Verify event-driven communication works correctly

**Example Backend Integration Test:**
```python
def test_installer_triggers_psadt_generation():
    # Given a test event bus
    event_bus = EventBus()
    
    # And a test orchestration core
    orchestrator = OrchestrationCore(event_bus=event_bus)
    
    # And connected agents
    installer_agent = InstallerAnalysisAgent(event_bus=event_bus)
    psadt_agent = PSADTGenerationAgent(event_bus=event_bus)
    
    # Register agents with orchestrator
    orchestrator.register_agent(installer_agent)
    orchestrator.register_agent(psadt_agent)
    
    # When we publish an installer analysis completion event
    event_bus.publish(
        "installer.analysis.completed", 
        {
            "package_id": "test-package",
            "installer_type": "MSI",
            "silent_switches": "/qn /norestart",
            "confidence": 0.95
        }
    )
    
    # Then the PSADT generation agent should be triggered
    # and produce a script with the correct parameters
    psadt_agent_called = wait_for_event(event_bus, "psadt.generation.started")
    assert psadt_agent_called
    
    # And eventually a script should be generated
    script_generated = wait_for_event(event_bus, "psadt.generation.completed")
    assert script_generated
    assert "/qn /norestart" in script_generated.payload["script_content"]
```

**Example Frontend Integration Test:**
```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import PackageDetail from '../../pages/packages/PackageDetail';
import { getPackageById } from '../../api/packageApi';

// Mock the API
vi.mock('../../api/packageApi');

describe('PackageDetail page', () => {
  it('loads and displays package details', async () => {
    // Setup mocks
    getPackageById.mockResolvedValue({
      id: '123',
      name: 'Test Package',
      version: '1.0.0',
      status: 'completed'
    });
    
    // Create query client
    const queryClient = new QueryClient();
    
    // Render component with router params
    render(
      <QueryClientProvider client={queryClient}>
        <PackageDetail id="123" />
      </QueryClientProvider>
    );
    
    // Check loading state
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText('Test Package')).toBeInTheDocument();
      expect(screen.getByText('1.0.0')).toBeInTheDocument();
      expect(screen.getByText('completed')).toBeInTheDocument();
    });
    
    // Verify API was called correctly
    expect(getPackageById).toHaveBeenCalledWith('123');
  });
});
```

### End-to-End (E2E) Tests

- **Scope:** Test the entire system flow from an end-user perspective. Interact with the application through its UI or API to validate complete user journeys.
- **Tools:** Playwright for browser testing, Pytest for API E2E testing
- **Strategy:** 
  - Define key user journeys to test
  - Setup test environment with necessary dependencies
  - Mock external services where appropriate
  - Execute full workflows from UI or API
  - Validate expected outcomes
- **Location:** 
  - Browser Tests: `e2e/`
  - API Tests: `backend/tests/e2e/`
- **Expectations:** 
  - Cover critical user journeys
  - Slower execution (may take minutes per test)
  - Run less frequently (nightly or before releases)
  - Focus on user-facing functionality

**Example E2E Test:**
```typescript
import { test, expect } from '@playwright/test';

test('complete packaging workflow', async ({ page }) => {
  // Start from dashboard
  await page.goto('/');
  
  // Click create new package button
  await page.getByRole('button', { name: 'Create Package' }).click();
  
  // Fill in package details
  await page.getByLabel('Name').fill('Test Application');
  await page.getByLabel('Version').fill('1.0.0');
  await page.getByLabel('Publisher').fill('Test Publisher');
  
  // Upload installer file
  await page.setInputFiles('input[type="file"]', './test-data/sample.msi');
  
  // Submit form
  await page.getByRole('button', { name: 'Create' }).click();
  
  // Wait for analysis to complete
  await page.waitForSelector('text=Analysis Complete', { timeout: 60000 });
  
  // Verify PSADT script generation started
  await page.waitForSelector('text=Generating PSADT Script', { timeout: 30000 });
  
  // Wait for WDAC policy creation
  await page.waitForSelector('text=Creating WDAC Policy', { timeout: 60000 });
  
  // Wait for final approval stage
  await page.waitForSelector('text=Waiting for Approval', { timeout: 60000 });
  
  // Check visualization is displayed
  await expect(page.getByTestId('decision-tree')).toBeVisible();
  
  // Approve package
  await page.getByRole('button', { name: 'Approve' }).click();
  
  // Verify package is completed
  await page.waitForSelector('text=Package Complete', { timeout: 30000 });
  
  // Navigate to package list
  await page.getByRole('link', { name: 'Packages' }).click();
  
  // Verify package is in the list
  await expect(page.getByText('Test Application')).toBeVisible();
});
```

### AI Agent Testing

- **Scope:** Specifically test AI agent decision-making quality, model interactions, and confidence scoring.
- **Tools:** Pytest, custom evaluation metrics
- **Strategy:** 
  - Test against known-good examples
  - Measure decision quality with quality metrics
  - Compare results against benchmarks
  - Test with different input variations
  - Verify confidence scoring correlates with accuracy
- **Location:** `backend/tests/agents/`
- **Expectations:** 
  - Verify decision quality meets or exceeds benchmarks
  - Test with diverse inputs for robustness
  - Confirm confidence scoring is accurate
  - Check explanation generation quality

**Example AI Agent Test:**
```python
def test_psadt_agent_script_quality():
    # Given a test agent with test model
    agent = PSADTGenerationAgent()
    
    # And a benchmark installer result
    installer_result = InstallerAnalysisResult(
        installer_type="MSI",
        product_name="Test Application",
        product_version="1.0.0",
        silent_switches="/qn /norestart",
        architecture="x64"
    )
    
    # When we generate a script
    script_result = agent.generate_script(installer_result)
    
    # Then the script should match quality metrics
    quality_score = evaluate_script_quality(script_result.script_content)
    assert quality_score >= 0.8  # 80% quality score minimum
    
    # And should contain expected elements
    assert "Test Application" in script_result.script_content
    assert "1.0.0" in script_result.script_content
    assert "/qn /norestart" in script_result.script_content
    assert "x64" in script_result.script_content
    
    # And confidence should correlate with quality
    assert abs(script_result.confidence_score - quality_score) < 0.2
```

### Visualization Testing

- **Scope:** Test the accuracy and usability of explainable AI visualizations.
- **Tools:** Vitest, React Testing Library, Playwright
- **Strategy:** 
  - Unit test visualization components
  - Verify visualization accurately represents model decisions
  - Test interactive elements for usability
  - Ensure visualization is accessible
- **Location:** `frontend/src/__tests__/visualization/`
- **Expectations:** 
  - Visualizations should accurately reflect agent decisions
  - Interactive elements should function correctly
  - Visualizations should be accessible
  - Compare visualization content to known-good examples

**Example Visualization Test:**
```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import DecisionTreeVisualization from '../../components/visualization/DecisionTree';

describe('DecisionTreeVisualization', () => {
  it('accurately renders decision nodes from agent data', () => {
    // Given decision tree data from an agent
    const decisionData = {
      id: 'root',
      label: 'Installer Analysis',
      children: [
        {
          id: 'node1',
          label: 'File Type Detection',
          decision: 'MSI',
          confidence: 0.95
        },
        {
          id: 'node2',
          label: 'Silent Switch Analysis',
          decision: '/qn /norestart',
          confidence: 0.85
        }
      ]
    };
    
    // When we render the visualization
    render(<DecisionTreeVisualization data={decisionData} />);
    
    // Then it should accurately display the decision structure
    expect(screen.getByText('Installer Analysis')).toBeInTheDocument();
    expect(screen.getByText('File Type Detection')).toBeInTheDocument();
    expect(screen.getByText('Silent Switch Analysis')).toBeInTheDocument();
    expect(screen.getByText('MSI')).toBeInTheDocument();
    expect(screen.getByText('/qn /norestart')).toBeInTheDocument();
    
    // And confidence indicators should be displayed
    const confidenceIndicators = screen.getAllByTestId('confidence-indicator');
    expect(confidenceIndicators).toHaveLength(2);
  });
});
```

## Specialized Testing Types

### Performance Testing

- **Scope & Goals:** Verify system performance meets requirements for processing time and resource usage.
- **Tools:** Locust, custom benchmarking tools
- **Strategy:**
  - Measure processing time for different installer types
  - Test with varying file sizes and complexity
  - Monitor memory usage during model execution
  - Verify scaling characteristics
  - Test parallel processing capabilities
- **Metrics:**
  - Package processing time (target: <30 minutes)
  - Memory usage (target: <32GB)
  - Concurrent package processing capability
  - UI responsiveness under load

### Security Testing

- **Scope & Goals:** Verify system security for handling untrusted installers and protecting data.
- **Tools:** OWASP ZAP, dependency scanning tools, custom security tests
- **Strategy:**
  - Test handling of malicious installer files
  - Verify proper input validation
  - Check for common vulnerabilities
  - Ensure proper authentication and authorization
  - Verify secure handling of sensitive data
- **Focus Areas:**
  - Installer file processing
  - Authentication and authorization
  - API security
  - Dependency vulnerabilities
  - Secure configuration

### Accessibility Testing

- **Scope & Goals:** Ensure UI is accessible to users with disabilities (WCAG 2.1 AA compliance).
- **Tools:** Lighthouse, axe, manual testing
- **Strategy:**
  - Automated accessibility checks
  - Keyboard navigation testing
  - Screen reader compatibility
  - Color contrast verification
  - Focus management testing
- **Key Areas:**
  - Dashboard and main workflows
  - Explainable AI visualizations
  - Forms and interactive elements
  - Notifications and alerts

## Test Data Management

- **Test Installers:**
  - Maintain a library of test installer files in `data/test_installers/`
  - Include various installer types (MSI, EXE, MSIX, etc.)
  - Add examples of complex installers for edge case testing
  - Document expected analysis results for each test installer

- **Test Database:**
  - Use Supabase local development instance for testing
  - Maintain seed data for common test scenarios
  - Reset database between test runs or use isolated test data
  - Use database snapshots for complex test setups

- **Test Fixtures:**
  - Implement Pytest fixtures for common test setup
  - Create factory functions for test data generation
  - Use parameterized tests for testing with multiple inputs
  - Share fixtures between test types when appropriate

## CI/CD Integration

- **Pull Request Validation:**
  - Run unit tests for all pull requests
  - Run selected integration tests for key workflows
  - Perform dependency scanning
  - Run linting and code quality checks
  - Generate code coverage report

- **Nightly Builds:**
  - Run full test suite including E2E tests
  - Perform performance testing
  - Run security scans
  - Run accessibility checks
  - Generate comprehensive test reports

- **Release Testing:**
  - Run full test suite with extended E2E coverage
  - Perform thorough performance testing
  - Run complete security assessment
  - Verify documentation accuracy
  - Generate release quality report

## Test Coverage Metrics

- **Code Coverage Targets:**
  - Overall Coverage: >70%
  - Core Components: >80%
  - AI Agents: >80%
  - API Endpoints: >90%
  - UI Components: >70%

- **Workflow Coverage:**
  - Each critical user journey tested end-to-end
  - Coverage of all installer types and scenarios
  - Testing of all error handling paths
  - Verification of all visualization types

## Change Log

| Change        | Date       | Version | Description                 | Author         |
| ------------- | ---------- | ------- | --------------------------- | -------------- |
| Initial draft | 2025-05-08 | 0.1     | Initial testing strategy    | Architect Agent |
