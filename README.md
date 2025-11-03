# Snippetbox

A web application for creating and sharing code snippets with user authentication and session management.

![Snippetbox Homepage](docs/images/home.jpeg)

## Prerequisites

- Go 1.20+
- MySQL 8.0

## Setup

### Database

Start MySQL and create the database schema:

```sql
CREATE DATABASE snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'web'@'localhost' IDENTIFIED BY 'your_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';
FLUSH PRIVILEGES;

USE snippetbox;

CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL,
    INDEX idx_snippets_created (created)
);

CREATE TABLE users (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    hashed_password CHAR(60) NOT NULL,
    created DATETIME NOT NULL,
    CONSTRAINT users_uc_email UNIQUE (email)
);

CREATE TABLE sessions (
    token CHAR(43) PRIMARY KEY,
    data BLOB NOT NULL,
    expiry TIMESTAMP(6) NOT NULL,
    INDEX sessions_expiry_idx (expiry)
);
```

### TLS Certificates

Generate self-signed certificates for local development:

```bash
mkdir -p tls
cd tls
go run $(go env GOROOT)/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost
cd ..
```

## Running

```bash
go run ./cmd/web
```

The server starts on `https://localhost:4000` by default.

### Configuration

```bash
go run ./cmd/web -addr=":9999" -dsn="web:password@/snippetbox?parseTime=true"
```

**Flags:**

- `-addr` - HTTP server address (default: `:4000`)
- `-dsn` - MySQL DSN (default: `web:pass@/snippetbox?parseTime=true`)

## Project Structure

```
snippetbox/
├── cmd/web/              # Application entry point
│   ├── main.go           # Server setup and configuration
│   ├── handlers.go       # HTTP request handlers
│   ├── routes.go         # Route definitions
│   ├── middleware.go     # HTTP middleware
│   ├── helpers.go        # Helper functions
│   └── templates.go      # Template caching
├── internal/
│   ├── models/           # Database models
│   └── validator/        # Form validation
├── ui/
│   ├── html/             # HTML templates
│   ├── static/           # CSS, JS, images
│   └── efs.go            # Embedded file system
└── tls/                  # TLS certificates (not committed)
```

## Security

- Bcrypt password hashing (cost 12)
- TLS 1.3 with strong cipher suites
- MySQL session storage with 12-hour lifetime
- Content Security Policy headers
- CSRF protection
- SQL injection prevention via prepared statements
- XSS protection headers

## Dependencies

```go
github.com/go-sql-driver/mysql        // MySQL driver
github.com/justinas/alice             // Middleware chaining
github.com/alexedwards/scs/v2         // Session management
github.com/alexedwards/scs/mysqlstore // MySQL session store
github.com/go-playground/form/v4      // Form decoding
golang.org/x/crypto/bcrypt            // Password hashing
```
