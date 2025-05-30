# Contributing to WuffChat

Thank you for your interest in contributing to WuffChat! This guide will help you get started.

## 🏷️ Naming Conventions

### Product vs Technical Names
- **WuffChat**: Use in user-facing content, documentation titles, and marketing materials
- **DogBot**: Use in code, repository names, and technical references

### Code Style
- **Python**: Follow PEP 8, use type hints
- **JavaScript/React**: Use ESLint configuration, functional components with hooks
- **API Endpoints**: Use kebab-case (e.g., `/start-conversation`)
- **Database Fields**: Use snake_case

## 🔄 Development Workflow

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/your-feature`
3. **Make** your changes
4. **Test** thoroughly
5. **Commit** with clear messages
6. **Push** to your fork
7. **Submit** a pull request

## 💬 Commit Messages

Follow conventional commits format:
```
type(scope): description

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `chore`: Maintenance tasks

## 🧪 Testing

Before submitting:
- Run all tests: `pytest` (backend) or `npm test` (frontend)
- Ensure no linting errors
- Test your changes manually
- Add tests for new features

## 📝 Pull Request Guidelines

- Link related issues
- Provide clear description of changes
- Include screenshots for UI changes
- Ensure CI/CD passes
- Request review from maintainers

## 🐛 Reporting Issues

Use GitHub Issues to report bugs:
- Search existing issues first
- Use issue templates
- Provide reproduction steps
- Include environment details

## 💡 Feature Requests

We welcome feature suggestions! Please:
- Check the roadmap in README
- Search existing requests
- Explain the use case
- Describe expected behavior

## 🌐 Localization

Currently, WuffChat is German-only, but we plan to add more languages. If you'd like to help with translations, please open an issue first.

## 📜 Code of Conduct

- Be respectful and inclusive
- Welcome newcomers
- Focus on constructive feedback
- Remember: we're all here to help dogs and their humans!

## 🤝 Getting Help

- Read the [main README](./README.md) first
- Check existing issues and PRs
- Join discussions in issues
- Ask questions - we're here to help!

Thank you for helping make WuffChat better! 🐕❤️