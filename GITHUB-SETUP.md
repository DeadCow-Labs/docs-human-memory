# Deploying to Vercel via GitHub

This guide walks you through deploying your Memory SDK documentation to Vercel using GitHub.

## Step 1: Push Your Documentation to GitHub

1. If you don't already have a GitHub repository for this project, create one:
   ```bash
   # Initialize git (if not already done)
   git init
   
   # Add all files to git
   git add .
   
   # Commit the changes
   git commit -m "Initial commit of Memory SDK documentation"
   
   # Create a new repository on GitHub.com and then add it as remote
   git remote add origin https://github.com/your-username/your-repo-name.git
   
   # Push to GitHub
   git push -u origin main  # or 'master' depending on your default branch
   ```

2. If you only want to push the docs-md folder to a new repository:
   ```bash
   # Create a new repository just for docs
   mkdir temp-docs-repo
   cd temp-docs-repo
   git init
   
   # Copy docs-md folder into this new repository
   cp -r ../docs-md/* .
   
   # Add all files and commit
   git add .
   git commit -m "Add Memory SDK documentation"
   
   # Add remote and push
   git remote add origin https://github.com/your-username/memory-sdk-docs.git
   git push -u origin main
   ```

## Step 2: Connect Vercel to GitHub

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. If you haven't already, sign up for a Vercel account
3. Click "Add New..." → "Project"
4. Choose "Continue with GitHub" and authorize Vercel

## Step 3: Import Your Repository

1. Vercel will show a list of your GitHub repositories
2. Find and select your documentation repository
3. Configure deployment settings:
   - Framework Preset: Select "Other"
   - Root Directory: 
     - If you pushed the entire project, set this to "docs-md"
     - If you created a repository just for the docs, leave as "./"
   - Build Command: Leave empty (no build step needed for Docsify)
   - Output Directory: Leave as "./" (default)

## Step 4: Deploy Your Documentation

1. Click "Deploy"
2. Vercel will build and deploy your documentation
3. When complete, you'll get a URL like `memory-sdk-docs.vercel.app`

## Step 5: Set Up Auto-Deployments (Optional)

By default, Vercel will automatically deploy when you push changes to your GitHub repository. If you want to modify this:

1. Go to your project in the Vercel dashboard
2. Click on "Settings" → "Git"
3. Under "Production Branch", you can change which branch triggers deployments
4. Under "Ignored Build Step", you can add conditions for when to skip deployments

## Step 6: Add a Custom Domain (Optional)

1. In your Vercel project, go to "Settings" → "Domains"
2. Click "Add" and enter your domain name
3. Follow the instructions to configure your DNS settings

## Troubleshooting

- **404 Errors**: Make sure the root directory is set correctly in Vercel
- **Styling Issues**: Check that all paths in index.html are correct
- **Missing Content**: Verify that the sidebar correctly links to your markdown files 