# Vectorcache Documentation

Official documentation for Vectorcache - AI-powered semantic caching for LLM applications.

## 🚀 Quick Start

Visit the live documentation: [docs.vectorcache.com](https://docs.vectorcache.com)

## 📚 What's Inside

- **Getting Started** - Quick start guide, installation, and configuration
- **SDK Documentation** - JavaScript/TypeScript, Python, and cURL examples
- **API Reference** - Complete API documentation
- **Guides** - Best practices, similarity tuning, cost optimization, and security
- **FAQ & Support** - Common questions and support resources

## 🛠️ Development

### Prerequisites

- Python 3.8+
- pip

### Local Setup

1. Clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/vectorcache-docs.git
cd vectorcache-docs
```

2. Install dependencies:
```bash
pip install mkdocs-material
pip install mkdocs-git-revision-date-localized-plugin
```

3. Run local server:
```bash
mkdocs serve
```

4. Open http://localhost:8000 in your browser

### Building

Build static site:
```bash
mkdocs build
```

Output will be in `site/` directory.

## 📝 Contributing

We welcome contributions! Here's how:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test locally: `mkdocs serve`
5. Commit: `git commit -m "Add your feature"`
6. Push: `git push origin feature/your-feature`
7. Open a Pull Request

### Documentation Structure

```
docs/
├── index.md                 # Home page
├── getting-started/         # Quick start, installation, config
├── sdk/                     # SDK documentation
├── api/                     # API reference
├── guides/                  # Best practices and guides
└── about/                   # FAQ, pricing, support
```

## 🚢 Deployment

Documentation is automatically deployed to GitHub Pages on push to `main` branch.

GitHub Actions workflow:
- Builds the docs with MkDocs Material
- Deploys to `gh-pages` branch
- Available at your GitHub Pages URL

## 📄 License

This documentation is licensed under [MIT License](LICENSE).

## 🆘 Support

- **Documentation Issues**: [GitHub Issues](https://github.com/YOUR_USERNAME/vectorcache-docs/issues)
- **Product Support**: support@vectorcache.com
- **Discord**: [Join our community](https://discord.gg/vectorcache)

## 🔗 Links

- [Main Website](https://vectorcache.com)
- [Dashboard](https://app.vectorcache.com)
- [Documentation](https://docs.vectorcache.com)
- [GitHub](https://github.com/YOUR_USERNAME/vectorcache)
