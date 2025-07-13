# AWS Amplify Angular Deployment Troubleshooting Guide

## Issue Summary
Your Angular app deployment is failing during the `npm ci` step in AWS Amplify. The logs show successful repository cloning and Node.js setup (20.19.0) but the build fails during dependency installation.

## Common Causes and Solutions

### 1. Node.js Version Compatibility Issues

**Problem**: Node.js 20.19.0 might have GLIBC compatibility issues with AWS Amplify's default Amazon Linux 2 build environment.

**Solution**: Create or update your `amplify.yml` with a specific Node version:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - nvm install 18.18.2
        - nvm use 18.18.2
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: dist/your-app-name  # Replace with your Angular app name
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

### 2. Alternative: Use Amazon Linux 2023 Build Image

If you need Node.js 20, update your build image settings:

1. Go to AWS Amplify Console
2. Navigate to `App settings` → `Build settings`
3. Scroll to `Build image settings` and click `Edit`
4. Select `Amazon Linux:2023` instead of the default Amazon Linux 2

### 3. Fix npm ci Issues

**Problem**: npm ci can fail due to package-lock.json inconsistencies or missing dependencies.

**Solutions**:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - nvm use 18.18.2
        - npm cache clean --force
        - rm -rf node_modules package-lock.json
        - npm install
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: dist/your-app-name
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

### 4. Angular-Specific Build Configuration

**Problem**: Angular apps often need specific build configurations for production deployment.

**Solution**: Update your `amplify.yml` for Angular:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - nvm use 18.18.2
        - npm ci
    build:
      commands:
        - npm run build -- --configuration=production
  artifacts:
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

### 5. Environment Variables and Dependencies

**Problem**: Missing environment variables or dev dependencies not being installed.

**Solution**: Ensure all required environment variables are set in Amplify Console and consider using npm install instead of npm ci:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - nvm use 18.18.2
        - npm install --production=false  # Install dev dependencies
    build:
      commands:
        - npm run build -- --configuration=production
  artifacts:
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

### 6. Debug Build Issues

**Problem**: Need to see the complete error logs.

**Solution**: Add debug commands to your `amplify.yml`:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - node --version
        - npm --version
        - cat package.json
        - nvm use 18.18.2
        - npm ci --verbose
    build:
      commands:
        - npm run build -- --configuration=production --verbose
  artifacts:
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

## Step-by-Step Resolution

### Step 1: Check Your Repository
Ensure your repository has:
- `package.json` with all required dependencies
- `package-lock.json` (if using npm ci)
- Proper Angular build scripts in package.json

### Step 2: Update amplify.yml
Create an `amplify.yml` file in your repository root with one of the configurations above.

### Step 3: Clear Amplify Cache
1. Go to AWS Amplify Console
2. Navigate to your app
3. Go to `App settings` → `Build settings`
4. Click `Actions` → `Clear cache and deploy`

### Step 4: Environment Variables
Ensure any required environment variables are set in:
- AWS Amplify Console → `App settings` → `Environment variables`

### Step 5: Monitor Build Logs
After deploying, monitor the complete build logs to identify specific error messages.

## Common Angular-Specific Issues

### Missing Angular CLI
If you're using Angular CLI commands:

```yaml
preBuild:
  commands:
    - nvm use 18.18.2
    - npm install -g @angular/cli
    - npm ci
```

### Build Output Directory
Make sure your `artifacts.baseDirectory` matches your Angular build output:
- Default: `dist/your-app-name`
- Check your `angular.json` file for the correct path

### Production Build
Always use production configuration for deployment:
```yaml
build:
  commands:
    - ng build --configuration=production
```

## If Issues Persist

1. **Check the complete build logs** in AWS Amplify Console for specific error messages
2. **Test locally** with the same Node.js version (18.18.2)
3. **Verify package.json** scripts work locally
4. **Consider using a Docker-based approach** if environment issues persist

## Next Steps

1. Update your repository with an appropriate `amplify.yml` file
2. Trigger a new deployment
3. Monitor the build logs for any remaining issues
4. If problems persist, share the complete error logs for further diagnosis