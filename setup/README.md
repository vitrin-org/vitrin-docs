# Vitrin Setup Guide

This directory contains setup and configuration files for the Vitrin platform.

## Environment Configuration

### Development Setup
- `env.example` - Development environment template
- `config.env.example` - Configuration template

### Production Setup  
- `env.production` - Production environment template

## Quick Start

1. Copy the appropriate environment file to your project root:
   ```bash
   cp setup/env.example .env
   ```

2. Update the values in `.env` according to your setup

3. Run the setup script:
   ```bash
   # Windows
   setup-dev.bat
   
   # Unix/Linux/Mac
   ./setup-dev.sh
   ```

## Documentation

- [Google OAuth Setup](GOOGLE_OAUTH_SETUP.md)
- [Testing Guide](TESTING_AND_REFACTORING_SUMMARY.md)
- [Testing Summary](TESTING_SUMMARY.md)
