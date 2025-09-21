# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

The **acquisitions** project is a Node.js/Express REST API built with modern JavaScript (ES modules) that provides user authentication functionality. It uses PostgreSQL with Drizzle ORM for data management, hosted on Neon database, and follows a clean MVC architectural pattern.

## Common Development Commands

### Development Server
- **Start development server**: `npm run dev` - Uses Node.js --watch for automatic restarts
- **Start production server**: `npm start`

### Code Quality
- **Lint code**: `npm run lint` - Check for ESLint issues
- **Fix linting issues**: `npm run lint:fix` - Auto-fix ESLint issues
- **Format code**: `npm run format` - Format with Prettier
- **Check formatting**: `npm run format:check` - Verify Prettier formatting

### Database Operations
- **Generate migrations**: `npm run db:generate` - Generate Drizzle migrations from schema changes
- **Run migrations**: `npm run db:migrate` - Apply pending migrations to database
- **Open database studio**: `npm run db:studio` - Launch Drizzle Studio for database inspection

### Environment Setup
Environment variables are configured in `.env`:
- `DATABASE_URL`: PostgreSQL connection string (Neon database)
- `PORT`: Server port (default: 3000)
- `NODE_ENV`: Environment mode (development/production)
- `LOG_LEVEL`: Logging level (info, debug, error)
- `JWT_SECRET`: Secret key for JWT tokens (change in production)

## Architecture Overview

### Project Structure
```
src/
├── config/          # Configuration files
│   ├── database.js  # Drizzle ORM database connection
│   └── logger.js    # Winston logger configuration
├── controllers/     # Request handlers and business logic
├── models/         # Drizzle ORM schema definitions
├── routes/         # Express route definitions
├── services/       # Business logic layer
├── utils/          # Utility functions (JWT, cookies, formatting)
└── validations/    # Zod schema validators
```

### Key Architectural Patterns

**MVC Architecture**: Clean separation between models (Drizzle schemas), views (JSON responses), and controllers

**Service Layer Pattern**: Business logic is encapsulated in service functions, keeping controllers thin and focused on HTTP concerns

**Path Mapping**: Uses Node.js import maps with `#` prefix for clean imports:
- `#config/*` → `./src/config/*`
- `#controllers/*` → `./src/controllers/*`
- `#models/*` → `./src/models/*`
- `#services/*` → `./src/services/*`
- `#utils/*` → `./src/utils/*`
- `#validations/*` → `./src/validations/*`

**Request Validation Flow**: 
1. Zod schemas validate incoming requests
2. Custom formatting utilities provide consistent error responses
3. Controllers handle HTTP-specific logic
4. Services contain pure business logic

### Database Layer
- **ORM**: Drizzle ORM with PostgreSQL
- **Connection**: Neon serverless PostgreSQL via `@neondatabase/serverless`
- **Migrations**: Schema changes in `src/models/*.js` → `drizzle-kit generate` → migrations in `drizzle/`
- **Schema Location**: Database schemas are defined in `src/models/` as Drizzle table definitions

### Authentication System
- **JWT-based**: Stateless authentication using JSON Web Tokens
- **Cookie Storage**: Tokens stored in HTTP-only cookies for security
- **Password Hashing**: bcrypt with salt rounds for secure password storage
- **Role-based**: Supports user roles (user/admin) for authorization

### Logging & Monitoring
- **Logger**: Winston with structured JSON logging
- **HTTP Logging**: Morgan middleware logs all requests
- **File Logging**: Separate error.lg and combined.log files in `logs/` directory
- **Console Logging**: Colorized console output in development

### Code Quality Standards
- **ES2022**: Modern JavaScript with ES modules
- **Linting**: ESLint with recommended rules and custom configurations
- **Formatting**: Prettier for consistent code style
- **Validation**: Zod for runtime type checking and validation

## Development Guidelines

### Adding New Routes
1. Create controller in `src/controllers/[feature].controller.js`
2. Create service functions in `src/services/[feature].service.js`
3. Add validation schema in `src/validations/[feature].validation.js`
4. Define routes in `src/routes/[feature].routes.js`
5. Register routes in `src/app.js`

### Database Schema Changes
1. Modify schema in `src/models/[table].model.js`
2. Run `npm run db:generate` to create migration
3. Review generated migration in `drizzle/` directory
4. Apply with `npm run db:migrate`

### Error Handling
- Use structured error responses via `formatValidationError` utility
- Log errors with Winston logger at appropriate levels
- Handle common cases (validation, authentication, database errors) consistently
- Use Express error handling middleware pattern

### Security Practices
- JWT secrets should be environment-specific
- Password hashing uses bcrypt with appropriate cost factor
- CORS and Helmet middleware for security headers
- Input validation on all endpoints using Zod schemas

### Testing New Features
- Health check endpoint available at `/health`
- API base endpoint at `/api` for connectivity testing
- Use Drizzle Studio (`npm run db:studio`) for database inspection during development