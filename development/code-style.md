# 📝 Development: Code Style Guide

<div align="center">

![Development](https://img.shields.io/badge/Development-Code%20Style-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan panduan penulisan kode untuk Chatbot Identifikasi Stunting, termasuk konvensi penamaan, format kode, dan best practices.

## 🎨 Code Formatting

### 1. TypeScript Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "lib": ["es2020"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 2. ESLint Configuration
```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module'
  },
  rules: {
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': ['error', { 'argsIgnorePattern': '^_' }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error'
  }
};
```

### 3. Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

## 📝 Naming Conventions

### 1. Files and Directories
```typescript
// File naming
user.service.ts        // Service files
user.controller.ts     // Controller files
user.model.ts         // Model files
user.interface.ts     // Interface files
user.type.ts          // Type definition files
user.test.ts          // Test files
user.util.ts          // Utility files

// Directory structure
src/
├── controllers/      // Controller files
├── services/         // Service files
├── models/          // Model files
├── interfaces/      // Interface files
├── types/           // Type definitions
├── utils/           // Utility functions
└── __tests__/       // Test files
```

### 2. Variables and Functions
```typescript
// Variables
const userCount: number = 0;
const isActive: boolean = true;
const userList: User[] = [];
const config: Config = {};

// Functions
function getUserById(id: string): Promise<User> {
  // ...
}

async function createUser(userData: UserInput): Promise<User> {
  // ...
}

// Classes
class UserService {
  private readonly userRepository: UserRepository;

  constructor(userRepository: UserRepository) {
    this.userRepository = userRepository;
  }

  public async findById(id: string): Promise<User> {
    // ...
  }
}
```

## 🏗️ Code Structure

### 1. Module Organization
```typescript
// user.module.ts
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { UserRepository } from './user.repository';

@Module({
  controllers: [UserController],
  providers: [UserService, UserRepository],
  exports: [UserService]
})
export class UserModule {}
```

### 2. Service Pattern
```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';
import { UserRepository } from './user.repository';
import { User } from './user.model';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User(dto);
    return this.userRepository.save(user);
  }

  async findById(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}
```

## 🔍 Code Quality

### 1. Type Safety
```typescript
// user.interface.ts
export interface User {
  id: string;
  name: string;
  phone: string;
  address: string;
  birthDate: Date;
  createdAt: Date;
  updatedAt: Date;
}

// user.dto.ts
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsString()
  @IsPhoneNumber()
  phone: string;

  @IsString()
  @IsNotEmpty()
  address: string;

  @IsDate()
  birthDate: Date;
}
```

### 2. Error Handling
```typescript
// error-handler.ts
export class AppError extends Error {
  constructor(
    public readonly message: string,
    public readonly statusCode: number = 500,
    public readonly code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// error.middleware.ts
export const errorHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      status: 'error',
      message: error.message,
      code: error.code
    });
  }

  console.error(error);
  return res.status(500).json({
    status: 'error',
    message: 'Internal server error'
  });
};
```

## 📚 Documentation

### 1. JSDoc Comments
```typescript
/**
 * User service for managing user operations
 */
export class UserService {
  /**
   * Create a new user
   * @param dto - User creation data
   * @returns Created user
   * @throws {AppError} If user with phone already exists
   */
  async createUser(dto: CreateUserDto): Promise<User> {
    // ...
  }

  /**
   * Find user by ID
   * @param id - User ID
   * @returns User if found
   * @throws {AppError} If user not found
   */
  async findById(id: string): Promise<User> {
    // ...
  }
}
```

### 2. README Files
```markdown
# User Module

## Overview
This module handles user management operations including registration,
profile updates, and user data retrieval.

## Features
- User registration
- Profile updates
- User data retrieval
- Phone number validation

## Usage
```typescript
import { UserService } from './user.service';

const userService = new UserService();
const user = await userService.createUser({
  name: 'John Doe',
  phone: '+6281234567890',
  address: 'Test Address'
});
```

## API
See [API Documentation](../api/README.md) for detailed endpoint information.
```

## 📚 Dokumentasi Terkait

- [Local Setup](local-setup.md)
- [Testing Guidelines](testing.md)
- [Production Guidelines](../deployment/production.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Testing Guidelines](testing.md) | [Lanjut ke Production Guidelines ➡️](../deployment/production.md)

</div> 