# ExpressJS with TypeScript - Modern Setup Guide

A comprehensive guide to creating a modern ExpressJS server with TypeScript using ES modules and contemporary building practices.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [TypeScript Configuration](#typescript-configuration)
- [Creating the Express Server](#creating-the-express-server)
- [Development Scripts](#development-scripts)
- [Building for Production](#building-for-production)
- [Best Practices](#best-practices)
- [Additional Resources](#additional-resources)

## Prerequisites

Before you begin, ensure you have the following installed:
- **Node.js** (v18 or higher recommended)
- **npm** or **yarn** package manager
- A code editor (VS Code recommended)

## Project Setup

### 1. Initialize Your Project

```bash
# Create a new directory for your project
mkdir express-ts-app
cd express-ts-app

# Initialize a new Node.js project
npm init -y
```

### 2. Install Dependencies

Install Express and other required dependencies:

```bash
# Production dependencies
npm install express

# Development dependencies
npm install -D typescript @types/node @types/express ts-node nodemon
```

**Dependency Breakdown:**
- `express`: The Express framework
- `typescript`: TypeScript compiler
- `@types/node`: Type definitions for Node.js
- `@types/express`: Type definitions for Express
- `ts-node`: Execute TypeScript files directly
- `nodemon`: Auto-restart server on file changes during development

### 3. Configure Package.json for ES Modules

Update your `package.json` to use ES modules:

```json
{
  "name": "express-ts-app",
  "version": "1.0.0",
  "type": "module",
  "description": "ExpressJS server with TypeScript",
  "main": "dist/index.js",
  "scripts": {
    "dev": "nodemon --exec ts-node --esm src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "type-check": "tsc --noEmit"
  },
  "keywords": ["express", "typescript", "nodejs"],
  "author": "",
  "license": "ISC"
}
```

## TypeScript Configuration

Create a `tsconfig.json` file in your project root with modern TypeScript settings:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022"],
    "moduleResolution": "node",
    "rootDir": "./src",
    "outDir": "./dist",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**Key Configuration Points:**
- `"type": "module"` in package.json enables ES modules
- `"module": "ES2022"` uses modern JavaScript modules
- `"strict": true` enables all strict type-checking options
- `noUnusedLocals` and `noUnusedParameters` enforce clean code (can be disabled during early development if too restrictive)
- Source files in `src/`, compiled output in `dist/`

## Creating the Express Server

### Project Structure

Create the following directory structure:

```
express-ts-app/
├── src/
│   ├── index.ts
│   ├── routes/
│   │   └── userRoutes.ts
│   ├── controllers/
│   │   └── userController.ts
│   ├── middleware/
│   │   └── errorHandler.ts
│   └── types/
│       └── express.d.ts
├── dist/
├── node_modules/
├── package.json
├── tsconfig.json
└── .gitignore
```

### 1. Create the Main Server File

Create `src/index.ts`:

```typescript
import express from 'express';
import type { Request, Response, Express } from 'express';
import dotenv from 'dotenv'

dotenv.config()
const app: Express = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/', (req: Request, res: Response) => {
  res.json({ message: 'Welcome to Express with TypeScript!' });
});


// Start server
app.listen(PORT, () => {
  console.log(`⚡️[server]: Server is running at http://localhost:${PORT}`);
});

export default app;
```

**Important:** Note the `.js` extension in import statements. This is required for ES modules even when importing TypeScript files.

### 2. Create Routes

Create `src/routes/userRoutes.ts`:

```typescript
import { Router } from 'express';
import { getUsers, getUserById, createUser } from '../controllers/userController.js';

const router = Router();

router.get('/', getUsers);
router.get('/:id', getUserById);
router.post('/', createUser);

export default router;
```

### 3. Create Controllers

Create `src/controllers/userController.ts`:

```typescript
import { Request, Response, NextFunction } from 'express';

interface User {
  id: number;
  name: string;
  email: string;
}

// Mock data
const users: User[] = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];

export const getUsers = (req: Request, res: Response, next: NextFunction): void => {
  try {
    res.json(users);
  } catch (error) {
    next(error);
  }
};

export const getUserById = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const id = parseInt(req.params.id);
    const user = users.find(u => u.id === id);
    
    if (!user) {
      res.status(404).json({ error: 'User not found' });
      return;
    }
    
    res.json(user);
  } catch (error) {
    next(error);
  }
};

export const createUser = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const newUser: User = {
      id: users.length + 1,
      name: req.body.name,
      email: req.body.email
    };
    
    users.push(newUser);
    res.status(201).json(newUser);
  } catch (error) {
    next(error);
  }
};
```

### 4. Create Middleware

Create `src/middleware/errorHandler.ts`:

```typescript
import { Request, Response, NextFunction } from 'express';

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  console.error(err.stack);
  
  res.status(500).json({
    error: 'Something went wrong!',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
};
```

### 5. Create .gitignore

Create `.gitignore` in the root directory:

```
node_modules/
dist/
.env
*.log
.DS_Store
```

## Development Scripts

### Running in Development Mode

```bash
npm run dev
```

This starts the server with hot-reloading. Any changes to your TypeScript files will automatically restart the server.

### Type Checking

```bash
npm run type-check
```

This runs the TypeScript compiler without emitting files, useful for checking types before building.

### Building for Production

```bash
npm run build
```

This compiles your TypeScript code to JavaScript in the `dist/` directory.

### Running Production Build

```bash
npm start
```

This runs the compiled JavaScript from the `dist/` directory.

## Building for Production

### Environment Variables

Create a `.env` file for environment-specific configuration:

```bash
npm install dotenv
```

Create `src/config/env.ts`:

```typescript
import dotenv from 'dotenv';

dotenv.config();

export const config = {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  // Add other environment variables here
};
```

### Production Deployment Steps

1. **Build the project:**
   ```bash
   npm run build
   ```

2. **Set environment variables:**
   ```bash
   export NODE_ENV=production
   export PORT=8080
   ```

3. **Start the server:**
   ```bash
   npm start
   ```

## Best Practices

### 1. **Use ES Modules Imports with .js Extensions**
   - Always add `.js` extension to relative imports, even for TypeScript files
   - Example: `import { something } from './module.js'`

### 2. **Leverage TypeScript's Type System**
   - Define interfaces for your data structures
   - Use strict mode for better type safety
   - Avoid `any` type when possible

### 3. **Organize Code by Feature**
   ```
   src/
   ├── features/
   │   ├── users/
   │   │   ├── user.controller.ts
   │   │   ├── user.routes.ts
   │   │   ├── user.service.ts
   │   │   └── user.types.ts
   │   └── products/
   │       ├── product.controller.ts
   │       └── ...
   ```

### 4. **Use Async/Await**
   - Modern async handling is cleaner than callbacks
   - Always handle errors with try-catch blocks

### 5. **Environment-Based Configuration**
   - Never hardcode sensitive data
   - Use environment variables for configuration
   - Keep `.env` out of version control

### 6. **Input Validation**
   ```bash
   npm install zod
   ```
   
   Use libraries like Zod for runtime type validation:
   ```typescript
   import { z } from 'zod';
   
   const userSchema = z.object({
     name: z.string().min(1),
     email: z.string().email()
   });
   ```

### 7. **Error Handling**
   - Create custom error classes
   - Use middleware for centralized error handling
   - Log errors appropriately

### 8. **Security Best Practices**
   ```bash
   npm install helmet cors
   ```
   
   ```typescript
   import helmet from 'helmet';
   import cors from 'cors';
   
   app.use(helmet());
   app.use(cors());
   ```

## Additional Resources

### Recommended Packages
- **Validation:** `zod`, `joi`, or `express-validator`
- **Database:** `prisma`, `typeorm`, or `mongoose`
- **Authentication:** `jsonwebtoken`, `passport`
- **Testing:** `jest`, `supertest`, `vitest`
- **Logging:** `winston`, `pino`
- **API Documentation:** `swagger-ui-express`

### Learning Resources
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Express.js Documentation](https://expressjs.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

### Example Testing Setup

Install testing dependencies:
```bash
npm install -D jest @types/jest ts-jest supertest @types/supertest
```

Create `jest.config.js`:
```javascript
export default {
  preset: 'ts-jest/presets/default-esm',
  testEnvironment: 'node',
  extensionsToTreatAsEsm: ['.ts'],
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1',
  },
  transform: {
    '^.+\\.tsx?$': ['ts-jest', {
      useESM: true,
    }],
  },
};
```

## Troubleshooting

### Common Issues

**Issue:** `Cannot find module` errors
- **Solution:** Ensure you're using `.js` extension in imports for ES modules

**Issue:** `ERR_UNKNOWN_FILE_EXTENSION`
- **Solution:** Make sure `"type": "module"` is in package.json

**Issue:** TypeScript errors with Express types
- **Solution:** Install `@types/express` and `@types/node`

## License

MIT

---

**Happy Coding! 🚀**

For questions or contributions, please open an issue or pull request.
