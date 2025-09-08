# Contributing to Vitrin

Thank you for your interest in contributing to Vitrin! This document provides guidelines and information for contributors.

## 📋 **Table of Contents**

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Process](#development-process)
- [Repository Structure](#repository-structure)
- [Coding Standards](#coding-standards)
- [Testing](#testing)
- [Documentation](#documentation)
- [Pull Request Process](#pull-request-process)
- [Issue Reporting](#issue-reporting)

## 🤝 **Code of Conduct**

This project adheres to a code of conduct. By participating, you are expected to uphold this code. Please report unacceptable behavior to the maintainers.

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

## 🚀 **Getting Started**

### Prerequisites

- Node.js 18+ (for frontend development)
- Python 3.8+ (for backend development)
- Docker and Docker Compose (for infrastructure)
- Git

### Fork and Clone

1. Fork the repository you want to contribute to
2. Clone your fork locally:
   ```bash
   git clone https://github.com/your-username/vitrin-[component].git
   cd vitrin-[component]
   ```

3. Add the upstream repository:
   ```bash
   git remote add upstream https://github.com/your-org/vitrin-[component].git
   ```

### Development Setup

Follow the setup instructions in each repository's README:

- [Backend Setup](https://github.com/your-org/vitrin-backend#quick-start)
- [Frontend Setup](https://github.com/your-org/vitrin-frontend#quick-start)
- [Infrastructure Setup](https://github.com/your-org/vitrin-infrastructure#quick-start)

## 🔄 **Development Process**

### Branch Naming

Use descriptive branch names with prefixes:

- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `refactor/` - Code refactoring
- `test/` - Test improvements
- `chore/` - Maintenance tasks

Examples:
- `feature/user-authentication`
- `fix/login-validation-error`
- `docs/api-documentation-update`

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

Examples:
```
feat(auth): add Google OAuth integration
fix(api): resolve product creation validation error
docs(readme): update installation instructions
```

## 🏗️ **Repository Structure**

### Backend (`vitrin-backend`)
```
backend/
├── core/                 # Core Django app
├── products/            # Product management
├── users/               # User management
├── vitrin/              # Django project settings
├── requirements.txt     # Python dependencies
└── manage.py           # Django management script
```

### Frontend (`vitrin-frontend`)
```
frontend/
├── src/
│   ├── components/      # React components
│   ├── services/        # API services
│   ├── hooks/          # Custom React hooks
│   ├── utils/          # Utility functions
│   └── types/          # TypeScript types
├── package.json        # Node.js dependencies
└── vite.config.ts      # Vite configuration
```

### Infrastructure (`vitrin-infrastructure`)
```
infrastructure/
├── docker-compose.yml   # Development environment
├── docker-compose.prod.yml # Production environment
├── nginx/              # Nginx configuration
└── scripts/            # Deployment scripts
```

### Shared (`vitrin-shared`)
```
shared/
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── constants/          # Application constants
└── package.json        # NPM package configuration
```

## 📝 **Coding Standards**

### TypeScript/JavaScript

- Use TypeScript for all new code
- Follow ESLint configuration
- Use Prettier for code formatting
- Prefer functional components with hooks
- Use meaningful variable and function names

```typescript
// Good
const handleUserLogin = async (credentials: LoginCredentials): Promise<void> => {
  try {
    await authService.login(credentials);
    navigate('/dashboard');
  } catch (error) {
    showError('Login failed');
  }
};

// Bad
const login = async (data: any) => {
  // Implementation
};
```

### Python

- Follow PEP 8 style guide
- Use type hints for function parameters and return values
- Write docstrings for all functions and classes
- Use meaningful variable names

```python
# Good
def create_product(
    title: str,
    description: str,
    category_id: int,
    author: User
) -> Product:
    """
    Create a new product.
    
    Args:
        title: Product title
        description: Product description
        category_id: Category ID
        author: Product author
        
    Returns:
        Created product instance
        
    Raises:
        ValidationError: If validation fails
    """
    # Implementation

# Bad
def create(data):
    # Implementation
```

### CSS/Styling

- Use Tailwind CSS utility classes
- Follow BEM methodology for custom CSS
- Use CSS variables for theming
- Mobile-first responsive design

```css
/* Good */
.product-card {
  @apply bg-white rounded-lg shadow-md p-4 hover:shadow-lg transition-shadow;
}

.product-card__title {
  @apply text-lg font-semibold text-gray-900 mb-2;
}

/* Bad */
.card {
  background: white;
  border-radius: 8px;
  /* ... */
}
```

## 🧪 **Testing**

### Frontend Testing

- Write unit tests for components and utilities
- Use React Testing Library for component testing
- Test user interactions and edge cases
- Maintain test coverage above 80%

```typescript
// Example test
import { render, screen, fireEvent } from '@testing-library/react';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('should submit form with valid credentials', async () => {
    const mockOnSubmit = jest.fn();
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    
    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });
});
```

### Backend Testing

- Write unit tests for models, views, and utilities
- Use Django's testing framework
- Test API endpoints with various scenarios
- Use factories for test data generation

```python
# Example test
from django.test import TestCase
from django.urls import reverse
from rest_framework.test import APITestCase
from rest_framework import status
from .models import Product
from .factories import ProductFactory, UserFactory

class ProductAPITest(APITestCase):
    def setUp(self):
        self.user = UserFactory()
        self.client.force_authenticate(user=self.user)
    
    def test_create_product(self):
        data = {
            'title': 'Test Product',
            'description': 'Test Description',
            'category': 1
        }
        response = self.client.post(reverse('product-list'), data)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Product.objects.count(), 1)
```

## 📚 **Documentation**

### Code Documentation

- Write clear docstrings for functions and classes
- Include type hints and parameter descriptions
- Document complex algorithms and business logic
- Keep README files up-to-date

### API Documentation

- Document all API endpoints
- Include request/response examples
- Document error codes and messages
- Keep OpenAPI/Swagger specs updated

## 🔀 **Pull Request Process**

### Before Submitting

1. **Update your fork** with the latest changes:
   ```bash
   git fetch upstream
   git checkout main
   git merge upstream/main
   ```

2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes** following the coding standards

4. **Write tests** for your changes

5. **Update documentation** if needed

6. **Run tests** and ensure they pass:
   ```bash
   # Frontend
   npm test
   npm run lint
   
   # Backend
   python manage.py test
   flake8 .
   ```

### Pull Request Template

When creating a pull request, include:

- **Description**: What changes were made and why
- **Type**: Feature, bug fix, documentation, etc.
- **Testing**: How the changes were tested
- **Screenshots**: For UI changes
- **Breaking Changes**: Any breaking changes
- **Checklist**: Ensure all items are completed

### Review Process

1. **Automated checks** must pass (tests, linting, etc.)
2. **Code review** by maintainers
3. **Testing** in development environment
4. **Approval** from at least one maintainer
5. **Merge** by maintainers

## 🐛 **Issue Reporting**

### Bug Reports

When reporting bugs, include:

- **Description**: Clear description of the bug
- **Steps to Reproduce**: Detailed steps to reproduce
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happens
- **Environment**: OS, browser, version, etc.
- **Screenshots**: If applicable
- **Logs**: Error logs or console output

### Feature Requests

When requesting features, include:

- **Description**: Clear description of the feature
- **Use Case**: Why this feature is needed
- **Proposed Solution**: How you think it should work
- **Alternatives**: Other solutions considered
- **Additional Context**: Any other relevant information

## 🏷️ **Labels and Milestones**

We use labels to categorize issues and pull requests:

- `bug` - Something isn't working
- `enhancement` - New feature or request
- `documentation` - Improvements or additions to documentation
- `good first issue` - Good for newcomers
- `help wanted` - Extra attention is needed
- `priority: high` - High priority
- `priority: low` - Low priority
- `status: in progress` - Currently being worked on
- `status: needs review` - Ready for review

## 📞 **Getting Help**

If you need help:

1. Check the [documentation](https://github.com/your-org/vitrin-docs)
2. Search existing [issues](https://github.com/your-org/vitrin/issues)
3. Join our [Discord community](https://discord.gg/vitrin)
4. Create a new issue with the `question` label

## 🎉 **Recognition**

Contributors are recognized in:

- [Contributors list](https://github.com/your-org/vitrin/graphs/contributors)
- Release notes for significant contributions
- Community highlights

Thank you for contributing to Vitrin! 🚀
