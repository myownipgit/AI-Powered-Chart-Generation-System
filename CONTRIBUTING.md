# 🤝 Contributing to AI-Powered Chart Generation System

We welcome contributions from the community! This guide will help you get started with contributing to our project.

## 📋 Table of Contents

- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Contributing Guidelines](#contributing-guidelines)
- [Code Standards](#code-standards)
- [Testing Requirements](#testing-requirements)
- [Submitting Changes](#submitting-changes)
- [Issue Reporting](#issue-reporting)
- [Feature Requests](#feature-requests)
- [Community Guidelines](#community-guidelines)

## 🚀 Getting Started

### Prerequisites

Before contributing, ensure you have:

- **Git** 2.40+
- **Python** 3.11+
- **Node.js** 18+
- **Docker** 24.0+
- **Basic knowledge** of FastAPI, React, and PostgreSQL

### Areas for Contribution

We welcome contributions in these areas:

#### **🤖 AI/ML Development**
- Natural language processing improvements
- Intent classification accuracy
- New chart type detection
- Performance optimization

#### **🎨 Frontend Development**
- React component improvements
- UI/UX enhancements
- Mobile responsiveness
- Accessibility features

#### **⚙️ Backend Development**
- API endpoint development
- Database optimization
- Export functionality
- WebSocket features

#### **📚 Documentation**
- API documentation
- User guides
- Code examples
- Tutorial content

#### **🧪 Testing**
- Unit test coverage
- Integration tests
- Performance testing
- Security testing

#### **🌍 Localization**
- Translation support
- Multi-language features
- Cultural adaptations
- Regional customizations

## 💻 Development Setup

### 1. Fork & Clone

```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/AI-Powered-Chart-Generation-System.git
cd AI-Powered-Chart-Generation-System

# Add upstream remote
git remote add upstream https://github.com/myownipgit/AI-Powered-Chart-Generation-System.git
```

### 2. Environment Setup

```bash
# Copy environment file
cp .env.example .env

# Edit environment variables
nano .env
```

### 3. Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Development dependencies

# Install pre-commit hooks
pre-commit install

# Run database setup
python scripts/setup_database.py

# Start development server
python main.py
```

### 4. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

### 5. Run Tests

```bash
# Backend tests
cd backend
python -m pytest tests/ -v --cov

# Frontend tests
cd frontend
npm test

# Integration tests
docker-compose -f docker-compose.test.yml up
```

## 📝 Contributing Guidelines

### Workflow Process

1. **Check existing issues** before starting work
2. **Create an issue** for new features or bugs
3. **Fork** the repository
4. **Create a feature branch** from `develop`
5. **Make your changes** following our standards
6. **Write tests** for your changes
7. **Submit a pull request** to `develop` branch

### Branch Naming Convention

```bash
# Feature branches
feature/add-chart-type-radar
feature/improve-nlp-accuracy

# Bug fix branches
bugfix/fix-export-pdf-issue
bugfix/resolve-websocket-connection

# Documentation branches
docs/update-api-documentation
docs/add-deployment-guide

# Hotfix branches (for main branch)
hotfix/security-vulnerability-fix
```

### Commit Message Format

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```bash
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types:**
- `feat`: New features
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `build`: Build system changes
- `ci`: CI/CD changes
- `chore`: Maintenance tasks

**Examples:**
```bash
feat(api): add batch chart generation endpoint

fix(frontend): resolve chart rendering issue in Safari

docs(readme): update installation instructions

test(backend): add unit tests for NLP processor
```

## 🎯 Code Standards

### Python Code Standards

#### **Style Guide**
- Follow [PEP 8](https://pep8.org/) style guide
- Use [Black](https://black.readthedocs.io/) for code formatting
- Use [isort](https://pycqa.github.io/isort/) for import sorting
- Use [flake8](https://flake8.pycqa.org/) for linting

#### **Code Formatting**
```bash
# Format code
black backend/
isort backend/

# Check linting
flake8 backend/

# Type checking
mypy backend/
```

#### **Code Structure**
```python
"""
Module docstring describing the purpose.

This module provides functionality for chart generation using AI.
"""

from typing import List, Optional, Dict, Any
import logging

from fastapi import HTTPException
from pydantic import BaseModel

logger = logging.getLogger(__name__)


class ChartRequest(BaseModel):
    """Request model for chart generation."""
    
    prompt: str
    user_id: Optional[str] = None
    
    class Config:
        """Pydantic configuration."""
        
        schema_extra = {
            "example": {
                "prompt": "Top 5 sales by region",
                "user_id": "user_123"
            }
        }


async def generate_chart(request: ChartRequest) -> Dict[str, Any]:
    """
    Generate a chart from natural language prompt.
    
    Args:
        request: Chart generation request
        
    Returns:
        Chart configuration and data
        
    Raises:
        HTTPException: If chart generation fails
    """
    try:
        # Implementation here
        logger.info(f"Generating chart for prompt: {request.prompt}")
        return {"success": True}
    except Exception as e:
        logger.error(f"Chart generation failed: {e}")
        raise HTTPException(status_code=500, detail=str(e))
```

### JavaScript/TypeScript Standards

#### **Style Guide**
- Use [ESLint](https://eslint.org/) with Airbnb config
- Use [Prettier](https://prettier.io/) for code formatting
- Follow React best practices
- Use TypeScript for type safety

#### **Code Formatting**
```bash
# Format code
npm run format

# Check linting
npm run lint

# Type checking
npm run type-check
```

#### **React Component Structure**
```typescript
import React, { useState, useEffect } from 'react';
import { ChartRequest, ChartResponse } from '../types/chart';
import { generateChart } from '../services/api';

interface ChartGeneratorProps {
  onChartGenerated?: (chart: ChartResponse) => void;
  className?: string;
}

/**
 * Chart generator component for creating charts from natural language.
 */
export const ChartGenerator: React.FC<ChartGeneratorProps> = ({
  onChartGenerated,
  className = ''
}) => {
  const [prompt, setPrompt] = useState<string>('');
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();
    
    if (!prompt.trim()) {
      setError('Please enter a chart description');
      return;
    }

    setIsLoading(true);
    setError(null);

    try {
      const chart = await generateChart({ prompt });
      onChartGenerated?.(chart);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className={`chart-generator ${className}`}>
      <textarea
        value={prompt}
        onChange={(e) => setPrompt(e.target.value)}
        placeholder="Describe your chart..."
        disabled={isLoading}
        className="prompt-input"
      />
      
      <button type="submit" disabled={isLoading || !prompt.trim()}>
        {isLoading ? 'Generating...' : 'Generate Chart'}
      </button>
      
      {error && <div className="error-message">{error}</div>}
    </form>
  );
};

export default ChartGenerator;
```

## 🧪 Testing Requirements

### Backend Testing

#### **Test Structure**
```python
# tests/test_chart_generation.py
import pytest
from fastapi.testclient import TestClient
from unittest.mock import patch, Mock

from app.main import app
from app.models.chart import ChartRequest

client = TestClient(app)


class TestChartGeneration:
    """Test suite for chart generation functionality."""

    def test_generate_chart_success(self):
        """Test successful chart generation."""
        request_data = {
            "prompt": "Top 5 sales by region",
            "user_id": "test_user"
        }
        
        response = client.post("/api/v1/charts/generate", json=request_data)
        
        assert response.status_code == 200
        data = response.json()
        assert data["success"] is True
        assert "chart_id" in data
        assert "config" in data

    def test_generate_chart_invalid_prompt(self):
        """Test chart generation with invalid prompt."""
        request_data = {"prompt": ""}
        
        response = client.post("/api/v1/charts/generate", json=request_data)
        
        assert response.status_code == 422

    @patch('app.services.nlp_processor.analyze_prompt')
    def test_generate_chart_nlp_failure(self, mock_analyze):
        """Test chart generation when NLP processing fails."""
        mock_analyze.side_effect = Exception("NLP service unavailable")
        
        request_data = {"prompt": "Top 5 sales by region"}
        response = client.post("/api/v1/charts/generate", json=request_data)
        
        assert response.status_code == 500
```

#### **Testing Standards**
- **Unit Tests**: Test individual functions and methods
- **Integration Tests**: Test API endpoints and database interactions
- **Performance Tests**: Test response times and load handling
- **Security Tests**: Test authentication and authorization

### Frontend Testing

#### **Test Structure**
```typescript
// __tests__/ChartGenerator.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ChartGenerator } from '../components/ChartGenerator';
import * as api from '../services/api';

// Mock the API module
jest.mock('../services/api');
const mockGenerateChart = api.generateChart as jest.MockedFunction<typeof api.generateChart>;

describe('ChartGenerator', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders the chart generator form', () => {
    render(<ChartGenerator />);
    
    expect(screen.getByPlaceholderText('Describe your chart...')).toBeInTheDocument();
    expect(screen.getByText('Generate Chart')).toBeInTheDocument();
  });

  it('generates chart on form submission', async () => {
    const mockChart = {
      id: 'chart_123',
      config: { chart_type: 'bar' },
      data: []
    };
    
    mockGenerateChart.mockResolvedValue(mockChart);
    
    const onChartGenerated = jest.fn();
    render(<ChartGenerator onChartGenerated={onChartGenerated} />);
    
    const textarea = screen.getByPlaceholderText('Describe your chart...');
    const submitButton = screen.getByText('Generate Chart');
    
    await userEvent.type(textarea, 'Top 5 sales by region');
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(mockGenerateChart).toHaveBeenCalledWith({
        prompt: 'Top 5 sales by region'
      });
      expect(onChartGenerated).toHaveBeenCalledWith(mockChart);
    });
  });

  it('displays error message on API failure', async () => {
    mockGenerateChart.mockRejectedValue(new Error('API Error'));
    
    render(<ChartGenerator />);
    
    const textarea = screen.getByPlaceholderText('Describe your chart...');
    const submitButton = screen.getByText('Generate Chart');
    
    await userEvent.type(textarea, 'Invalid prompt');
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText('API Error')).toBeInTheDocument();
    });
  });
});
```

## 📥 Submitting Changes

### Pull Request Process

1. **Ensure your fork is up to date**
   ```bash
   git fetch upstream
   git checkout develop
   git merge upstream/develop
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes and commit**
   ```bash
   git add .
   git commit -m "feat: add new chart type support"
   ```

4. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

5. **Create a pull request** on GitHub

### Pull Request Template

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## How Has This Been Tested?
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Screenshots (if applicable)
Add screenshots to help explain your changes.

## Checklist
- [ ] My code follows the style guidelines of this project
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

### Review Process

1. **Automated Checks**: CI/CD pipeline runs tests and checks
2. **Code Review**: Maintainers review your code
3. **Feedback**: Address any requested changes
4. **Approval**: PR gets approved and merged

## 🐛 Issue Reporting

### Bug Reports

Use our [Bug Report Template](https://github.com/myownipgit/AI-Powered-Chart-Generation-System/issues/new?template=bug_report.md):

```markdown
**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected behavior**
A clear and concise description of what you expected to happen.

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Environment:**
 - OS: [e.g. iOS]
 - Browser [e.g. chrome, safari]
 - Version [e.g. 22]

**Additional context**
Add any other context about the problem here.
```

### Security Issues

For security vulnerabilities, please:

1. **Do NOT create a public issue**
2. **Email us** at security@chartgen.ai
3. **Include** detailed information about the vulnerability
4. **Wait** for our response before disclosing publicly

## 💡 Feature Requests

Use our [Feature Request Template](https://github.com/myownipgit/AI-Powered-Chart-Generation-System/issues/new?template=feature_request.md):

```markdown
**Is your feature request related to a problem? Please describe.**
A clear and concise description of what the problem is.

**Describe the solution you'd like**
A clear and concise description of what you want to happen.

**Describe alternatives you've considered**
A clear and concise description of any alternative solutions or features you've considered.

**Additional context**
Add any other context or screenshots about the feature request here.
```

## 🌟 Community Guidelines

### Code of Conduct

We are committed to providing a friendly, safe, and welcoming environment for all. Please read our [Code of Conduct](CODE_OF_CONDUCT.md).

### Communication Channels

- **GitHub Discussions**: [Project Discussions](https://github.com/myownipgit/AI-Powered-Chart-Generation-System/discussions)
- **Discord**: [Join our server](https://discord.gg/chartgen)
- **Email**: contributors@chartgen.ai
- **Twitter**: [@chartgen_ai](https://twitter.com/chartgen_ai)

### Recognition

Contributors are recognized in several ways:

- **Contributors page** on our website
- **Special badges** on GitHub
- **Monthly contributor highlights**
- **Invitation to contributor events**

### Getting Help

Need help? Here are your options:

1. **Check existing documentation** and issues first
2. **Join our Discord** for real-time help
3. **Create a GitHub discussion** for general questions
4. **Email us** for private inquiries

---

## 🙏 Thank You

Thank you for contributing to the AI-Powered Chart Generation System! Your contributions help make data visualization accessible to everyone.

### Top Contributors

<!-- This section will be automatically updated -->
Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification.

---

**Happy Contributing! 🚀**
