# Quick Start Guide

Get up and running with Vitrin in minutes!

## 🚀 **Prerequisites**

Before you begin, make sure you have the following installed:

- **Node.js** 18+ ([Download](https://nodejs.org/))
- **Python** 3.8+ ([Download](https://python.org/))
- **Docker** and **Docker Compose** ([Download](https://docker.com/))
- **Git** ([Download](https://git-scm.com/))

## ⚡ **Option 1: Docker (Recommended)**

The fastest way to get started is using Docker:

### 1. Clone the Infrastructure Repository

```bash
git clone https://github.com/your-org/vitrin-infrastructure.git
cd vitrin-infrastructure
```

### 2. Start the Development Environment

```bash
# Windows
docker-scripts.bat dev

# Linux/Mac
./docker-scripts.sh dev
```

### 3. Access the Application

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/api/docs/

That's it! The application should be running with all services.

## 🛠️ **Option 2: Manual Setup**

If you prefer to run services individually:

### 1. Clone All Repositories

```bash
# Create a workspace directory
mkdir vitrin-workspace
cd vitrin-workspace

# Clone all repositories
git clone https://github.com/your-org/vitrin-backend.git
git clone https://github.com/your-org/vitrin-frontend.git
git clone https://github.com/your-org/vitrin-infrastructure.git
git clone https://github.com/your-org/vitrin-shared.git
```

### 2. Setup Backend

```bash
cd vitrin-backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Setup environment
cp env.example .env
# Edit .env with your database and API keys

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Start the server
python manage.py runserver
```

### 3. Setup Frontend

```bash
cd vitrin-frontend

# Install dependencies
npm install

# Setup environment
cp env.example .env
# Edit .env with your API URL

# Start development server
npm run dev
```

### 4. Setup Database (Optional)

If you want to use PostgreSQL instead of SQLite:

```bash
# Using Docker
docker run --name vitrin-postgres \
  -e POSTGRES_DB=vitrin \
  -e POSTGRES_USER=vitrin \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  -d postgres:15

# Update your .env file with PostgreSQL settings
```

## 🔧 **Configuration**

### Environment Variables

#### Backend (.env)
```env
SECRET_KEY=your-secret-key-here
DEBUG=True
DATABASE_URL=postgresql://vitrin:password@localhost:5432/vitrin
REDIS_URL=redis://localhost:6379/0
GOOGLE_OAUTH_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH_CLIENT_SECRET=your-google-client-secret
```

#### Frontend (.env)
```env
VITE_API_URL=http://localhost:8000/api
VITE_GOOGLE_CLIENT_ID=your-google-client-id
```

### Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google+ API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URIs:
   - `http://localhost:3000/auth/google/callback`
   - `http://localhost:8000/auth/google/callback`
6. Copy Client ID and Client Secret to your .env files

## 🎯 **First Steps**

### 1. Create Your First Product

1. Go to http://localhost:3000
2. Sign up for an account
3. Click "Add Product"
4. Fill in the product details
5. Upload an image
6. Publish your product

### 2. Explore the API

Visit http://localhost:8000/api/docs/ to explore the API documentation.

### 3. Test Authentication

Try the authentication endpoints:
- `POST /api/auth/register/` - Register a new user
- `POST /api/auth/login/` - Login with credentials
- `POST /api/auth/google/` - Login with Google OAuth

## 🐛 **Troubleshooting**

### Common Issues

#### Port Already in Use
```bash
# Find process using port 3000
lsof -i :3000  # Mac/Linux
netstat -ano | findstr :3000  # Windows

# Kill the process
kill -9 <PID>  # Mac/Linux
taskkill /PID <PID> /F  # Windows
```

#### Database Connection Issues
- Check if PostgreSQL is running
- Verify database credentials in .env
- Ensure database exists

#### Frontend Build Issues
```bash
# Clear node modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

#### Backend Import Issues
```bash
# Activate virtual environment
source venv/bin/activate  # Mac/Linux
venv\Scripts\activate  # Windows

# Reinstall dependencies
pip install -r requirements.txt
```

### Getting Help

- Check the [FAQ](faq.md)
- Look at [Troubleshooting Guide](troubleshooting.md)
- Open an issue in the appropriate repository
- Join our [Discord community](https://discord.gg/vitrin)

## 🚀 **Next Steps**

Now that you have Vitrin running:

1. **Read the Documentation**: Explore the [API documentation](api/overview.md)
2. **Learn the Architecture**: Check out [System Architecture](architecture/system-architecture.md)
3. **Contribute**: See [Contributing Guidelines](../CONTRIBUTING.md)
4. **Deploy**: Follow the [Deployment Guide](deployment.md)

## 📚 **Additional Resources**

- [Installation Guide](installation.md) - Detailed installation instructions
- [Configuration Guide](configuration.md) - Environment and app configuration
- [Development Setup](development-setup.md) - Local development environment
- [API Documentation](api/overview.md) - Complete API reference

Happy coding! 🎉
