# Deploying to Vercel

This documentation site is built with Docsify and can be easily deployed to Vercel.

## Method 1: Deploy via GitHub

1. Push your docs-md folder to a GitHub repository
2. Go to [Vercel Dashboard](https://vercel.com/dashboard)
3. Click "New Project"
4. Import your GitHub repository
5. Set the following configurations:
   - Framework Preset: Other
   - Root Directory: docs-md
   - Build Command: None (leave empty)
   - Output Directory: ./ (default)
6. Click "Deploy"

## Method 2: Deploy via Vercel CLI

1. Install Vercel CLI:
```bash
npm install -g vercel
```

2. Navigate to your docs-md directory:
```bash
cd docs-md
```

3. Deploy to Vercel:
```bash
vercel
```

4. Follow the prompts:
   - Set up and deploy? Yes
   - Which scope? (Select your account)
   - Link to existing project? No
   - What's your project's name? memory-sdk-docs (or any name)
   - In which directory is your code located? ./ (current directory)
   - Want to override settings? No

## Configuration

The `vercel.json` file in this directory contains the necessary configuration for the deployment:

```json
{
  "version": 2,
  "name": "memory-sdk-docs",
  "builds": [
    {
      "src": "**/*",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ]
}
```

## Custom Domain

After deployment, you can add a custom domain in the Vercel dashboard:

1. Go to your project settings
2. Navigate to "Domains"
3. Add your domain and follow the instructions for DNS configuration 