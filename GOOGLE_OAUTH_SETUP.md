# Google OAuth Setup Guide

## 🔧 Backend Configuration

The backend Google authentication is already configured and working. The following dependencies are installed:
- `PyJWT==2.8.0` - For JWT token verification
- `google-auth==2.23.4` - For Google authentication
- `django-allauth==0.57.0` - For OAuth integration

## 🌐 Frontend Configuration

### Step 1: Get Google OAuth Client ID

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google+ API
4. Go to "Credentials" → "Create Credentials" → "OAuth 2.0 Client IDs"
5. Set application type to "Web application"
6. Add authorized origins:
   - `http://localhost:5173` (for development)
   - `http://localhost:3000` (for production build)
7. Copy the Client ID

### Step 2: Update Frontend Configuration

Replace `YOUR_GOOGLE_CLIENT_ID` in `src/components/LoginSignup.tsx` with your actual Google Client ID.

### Step 3: Environment Variables (Optional)

You can also use environment variables for better security:

1. Create `.env` file in the project root:
```env
VITE_GOOGLE_CLIENT_ID=your_actual_google_client_id_here
```

2. Update `src/components/LoginSignup.tsx`:
```typescript
client_id: import.meta.env.VITE_GOOGLE_CLIENT_ID || 'YOUR_GOOGLE_CLIENT_ID',
```

## 🚀 Quick Fix for Testing

For immediate testing, you can use a demo Google Client ID or set up your own:

1. **Option 1: Use a demo ID (limited functionality)**
   - Replace `YOUR_GOOGLE_CLIENT_ID` with a demo ID for testing

2. **Option 2: Set up your own (recommended)**
   - Follow the steps above to get your own Google Client ID
   - This will give you full functionality

## 🔍 Troubleshooting

### Common Issues:

1. **"Invalid client ID" error**
   - Make sure you're using the correct Client ID from Google Cloud Console
   - Ensure the domain is authorized in Google Cloud Console

2. **"Origin not allowed" error**
   - Add your domain to authorized origins in Google Cloud Console
   - For development: `http://localhost:5173`

3. **"API not enabled" error**
   - Enable Google+ API in Google Cloud Console
   - Make sure billing is enabled (required for some APIs)

### Testing the Setup:

1. Start the application: `npm run dev`
2. Go to the login page
3. Click "Sign in with Google"
4. You should see the Google sign-in popup
5. After successful authentication, you should be logged in

## 📝 Current Status

- ✅ Backend Google authentication is implemented and working
- ✅ Frontend Google Identity Services integration is ready
- ⚠️  Google Client ID needs to be configured
- ✅ JWT token handling is implemented
- ✅ User creation/login flow is working

Once you configure the Google Client ID, Google login should work perfectly!
