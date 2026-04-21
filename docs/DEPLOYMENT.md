# TutorNation Backend - Deployment Guide

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Current Infrastructure](#current-infrastructure)
3. [Database Connection Architecture](#database-connection-architecture)
4. [Deployment Process](#deployment-process)
5. [Configuration Management](#configuration-management)
6. [Troubleshooting](#troubleshooting)
7. [Migration to Production Database](#migration-to-production-database)

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────────┐
│                 │         │                  │         │                     │
│  iOS Client     │────────▶│  EC2 Instance    │────────▶│  Database           │
│  (TutorNation)  │         │  (Spring Boot)   │         │  (H2/PostgreSQL)    │
│                 │         │                  │         │                     │
└─────────────────┘         └──────────────────┘         └─────────────────────┘
                                    │
                                    │
                                    ▼
                            ┌──────────────────┐
                            │  Developer        │
                            │  Laptop           │
                            │  (Build & Deploy) │
                            └──────────────────┘
```

### Components

1. **EC2 Instance** - Hosts the Spring Boot application
2. **Database** - Stores application data (currently H2, should be PostgreSQL)
3. **Developer Machine** - Builds and deploys the application
4. **iOS Client** - Consumes the REST API

---

## Current Infrastructure

### EC2 Instance Details

**Location:** AWS us-east-2 (Ohio)
- **Public IP:** `18.118.212.178`
- **Operating System:** Amazon Linux 2023.10.20260302
- **Java Version:** OpenJDK 17.0.18 (Amazon Corretto)
- **Application Port:** `8080`
- **SSH User:** `ec2-user`
- **SSH Key:** `/Users/minseok/tutornation-prod-us-east-2-keypair.pem`

**Security Group Requirements:**
- Port 22 (SSH) - For deployment and management
- Port 8080 (HTTP) - For application API access
- Outbound internet access - For database connections

### Database Configuration

#### Current Setup (Development - H2)
```properties
Database: H2 In-Memory
URL: jdbc:h2:mem:testdb
Username: sa
Password: (empty)
```

**⚠️ IMPORTANT:** This is a temporary in-memory database. All data is lost when the application restarts.

#### Available Production Database (Supabase PostgreSQL)
```properties
Host: db.miicefbuykhjafjkhxgg.supabase.co
Port: 5432
Database: postgres
Username: postgres
Password: NpkPv5Cz9VrEXz2C
```

---

## Database Connection Architecture

### Why EC2 Needs a Database

The EC2 instance runs the Spring Boot application, which is stateless. This means:

1. **Application Logic** - Runs on EC2 (business rules, API endpoints, authentication)
2. **Data Storage** - Must be in a separate database (user data, tutoring sessions, etc.)
3. **Separation of Concerns** - EC2 can be restarted/replaced without losing data

### The EC2-Database Relationship

```
┌─────────────────────────────────────────────────────────────────┐
│                          EC2 Instance                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Spring Boot Application                      │  │
│  │                                                           │  │
│  │  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐  │  │
│  │  │ Controllers │   │   Services   │   │  Repositories│  │  │
│  │  │  (API)      │──▶│  (Business)  │──▶│   (Data)     │  │  │
│  │  └─────────────┘   └──────────────┘   └──────┬───────┘  │  │
│  │                                               │          │  │
│  └───────────────────────────────────────────────┼──────────┘  │
│                                                  │             │
└──────────────────────────────────────────────────┼─────────────┘
                                                   │
                                    JDBC Connection (TCP/IP)
                                                   │
                  ┌────────────────────────────────▼──────────────┐
                  │           Database Server                     │
                  │  ┌────────────────────────────────────────┐   │
                  │  │  PostgreSQL Database                   │   │
                  │  │  ┌──────────┐  ┌──────────┐           │   │
                  │  │  │  Users   │  │ Sessions │  ...      │   │
                  │  │  └──────────┘  └──────────┘           │   │
                  │  └────────────────────────────────────────┘   │
                  │                                               │
                  │  Options:                                     │
                  │  - AWS RDS (managed PostgreSQL)               │
                  │  - Supabase (hosted PostgreSQL)               │
                  │  - Self-hosted PostgreSQL on separate EC2     │
                  └───────────────────────────────────────────────┘
```

### Connection Flow

1. **Application Startup:**
   ```
   EC2 Application → Reads application.properties
                  → Initializes HikariCP Connection Pool
                  → Establishes DB connections (default: 2-10 connections)
                  → Validates database schema
                  → Application ready to serve requests
   ```

2. **API Request Processing:**
   ```
   iOS Client → HTTP Request to EC2:8080
             → Controller receives request
             → Service layer processes business logic
             → Repository requests data
             → Connection pool provides DB connection
             → SQL query executed on database
             → Results returned through layers
             → JSON response sent to client
   ```

### Why This Architecture?

**Benefits:**
1. **Data Persistence** - Data survives application restarts
2. **Scalability** - Can add multiple EC2 instances connecting to same database
3. **Backup & Recovery** - Database can be backed up independently
4. **Performance** - Optimized database server with connection pooling
5. **Security** - Database can be in private subnet, not publicly accessible

**Connection Pool (HikariCP):**
```properties
spring.datasource.hikari.maximum-pool-size=10    # Max connections
spring.datasource.hikari.minimum-idle=2          # Min idle connections
spring.datasource.hikari.connection-timeout=30000 # 30 seconds
spring.datasource.hikari.idle-timeout=600000     # 10 minutes
```

This means:
- EC2 maintains 2-10 persistent connections to the database
- Connections are reused for efficiency (avoiding overhead of creating new connections)
- If a connection is idle for 10 minutes, it's closed
- If all connections are busy, requests wait up to 30 seconds

---

## Deployment Process

### Prerequisites

1. **Local Development Machine:**
   - Java 17 or higher
   - Maven (included via `mvnw` wrapper)
   - SSH access to EC2 instance
   - SSH key file: `tutornation-prod-us-east-2-keypair.pem`

2. **EC2 Instance Setup:**
   - Amazon Linux 2023
   - Java 17 installed
   - Security group allowing ports 22 and 8080
   - SSH key pair configured

### Deployment Files Structure

```
TutorNation_Backend/
├── deploy.sh                        # Main deployment script (local)
├── set-env.sh                       # Environment variables (local)
├── src/main/resources/
│   └── application.properties       # Default application config
└── EC2: ~/tutornation-backend/
    ├── start.sh                     # Application startup script
    ├── stop.sh                      # Application shutdown script
    ├── application.properties       # Production config (overrides default)
    ├── TutorNation_Backend-*.jar    # Application JAR
    ├── app.log                      # Application logs
    └── app.pid                      # Process ID file
```

### Step-by-Step Deployment Process

#### 1. Build Phase (Local Machine)

```bash
./deploy.sh
```

**What happens:**
```bash
# 1. Clean previous builds
./mvnw clean

# 2. Compile Java code
./mvnw compile

# 3. Run tests (currently skipped with -DskipTests)
./mvnw test

# 4. Package into JAR
./mvnw package
# Creates: target/TutorNation_Backend-0.0.1-SNAPSHOT.jar (~73.5 MB)
```

**Build Output:**
- Executable JAR file containing:
  - Compiled Java classes
  - Dependencies (Spring Boot, PostgreSQL driver, etc.)
  - Resources (application.properties, static files)
  - Embedded Tomcat server

#### 2. Stop Current Application (EC2)

```bash
ssh -i tutornation-prod-us-east-2-keypair.pem ec2-user@18.118.212.178 \
    'cd ~/tutornation-backend && ./stop.sh'
```

**stop.sh Script:**
```bash
#!/bin/bash
cd ~/tutornation-backend
if [ -f app.pid ]; then
    PID=$(cat app.pid)          # Read process ID
    kill $PID 2>/dev/null       # Send SIGTERM to gracefully stop
    echo "Stopped application with PID: $PID"
    rm app.pid                  # Clean up PID file
else
    echo "No PID file found. Application may not be running."
fi
```

**Graceful Shutdown Process:**
1. Spring Boot receives SIGTERM signal
2. Stops accepting new requests
3. Completes in-flight requests (up to 30 seconds)
4. Closes database connections
5. Releases resources
6. Process exits

#### 3. Transfer JAR to EC2

```bash
scp -i tutornation-prod-us-east-2-keypair.pem \
    target/TutorNation_Backend-0.0.1-SNAPSHOT.jar \
    ec2-user@18.118.212.178:~/tutornation-backend/
```

**Transfer Details:**
- Protocol: SCP (Secure Copy Protocol over SSH)
- Speed: Depends on internet connection (~1-2 minutes for 73MB)
- Authentication: SSH key-based (no password needed)
- Overwrites existing JAR file

#### 4. Start New Application (EC2)

```bash
ssh -i tutornation-prod-us-east-2-keypair.pem ec2-user@18.118.212.178 \
    'cd ~/tutornation-backend && ./start.sh'
```

**start.sh Script:**
```bash
#!/bin/bash
cd ~/tutornation-backend
nohup java -jar TutorNation_Backend-0.0.1-SNAPSHOT.jar \
    --spring.config.location=file:./application.properties \
    > app.log 2>&1 &
echo $! > app.pid
echo "Application started with PID: $(cat app.pid)"
```

**Breakdown:**
- `nohup` - Keeps process running after SSH disconnects
- `java -jar` - Runs the executable JAR
- `--spring.config.location` - Uses local application.properties (overrides embedded one)
- `> app.log` - Redirects stdout to app.log
- `2>&1` - Redirects stderr to stdout (both go to app.log)
- `&` - Runs in background
- `$!` - Gets PID of background process
- Saves PID to app.pid for later stop operations

**Application Startup Sequence:**
```
1. JVM starts (allocates memory, loads classes)
2. Spring Boot initializes
3. Reads application.properties
4. Connects to database (HikariCP pool)
5. Initializes Spring Data JDBC repositories
6. Starts embedded Tomcat server on port 8080
7. Registers REST API endpoints
8. Application ready (typically 3-8 seconds)
```

### Complete deploy.sh Script Explained

```bash
#!/bin/bash
set -e  # Exit immediately if any command fails

# Configuration
EC2_HOST="18.118.212.178"
EC2_USER="ec2-user"
KEY_FILE="/Users/minseok/tutornation-prod-us-east-2-keypair.pem"
REMOTE_DIR="~/tutornation-backend"
JAR_FILE="target/TutorNation_Backend-0.0.1-SNAPSHOT.jar"

# Build
echo "🔨 Building application..."
./mvnw clean package -DskipTests
if [ ! -f "$JAR_FILE" ]; then
    echo "❌ Build failed - JAR file not found"
    exit 1
fi
echo "✅ Build successful"

# Stop
echo "🛑 Stopping application on EC2..."
ssh -i "$KEY_FILE" "$EC2_USER@$EC2_HOST" \
    "cd $REMOTE_DIR && ./stop.sh" || echo "App was not running"

# Transfer
echo "📦 Transferring JAR to EC2..."
scp -i "$KEY_FILE" "$JAR_FILE" "$EC2_USER@$EC2_HOST:$REMOTE_DIR/"

# Start
echo "🚀 Starting application on EC2..."
ssh -i "$KEY_FILE" "$EC2_USER@$EC2_HOST" \
    "cd $REMOTE_DIR && ./start.sh"

# Done
echo ""
echo "✅ Deployment complete!"
echo "📝 View logs: ssh -i $KEY_FILE $EC2_USER@$EC2_HOST 'tail -f $REMOTE_DIR/app.log'"
echo "🌐 Application URL: http://$EC2_HOST:8080"
```

---

## Configuration Management

### Configuration File Hierarchy

Spring Boot uses this priority order (highest to lowest):

1. **Command line arguments:** `--spring.datasource.url=...`
2. **External config file:** `--spring.config.location=file:./application.properties`
3. **Internal config file:** `src/main/resources/application.properties`
4. **Default values:** Spring Boot defaults

Current setup uses #2, which overrides #3.

### Local Development Configuration

**File:** `src/main/resources/application.properties`

```properties
spring.application.name=TutorNation_Backend
spring.docker.compose.enabled=false

# Connection Pool Settings
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000

# Flyway Database Migration
spring.flyway.enabled=false
spring.flyway.baseline-on-migrate=true
spring.flyway.locations=classpath:db/migration

# JPA/Hibernate Settings
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true

# Batch Settings
spring.batch.job.enabled=false

# AI Configuration
spring.autoconfigure.exclude=org.springframework.ai.model.anthropic.autoconfigure.AnthropicChatAutoConfiguration
```

**Key Settings:**
- `ddl-auto=validate` - Don't auto-create tables, just validate schema matches entities
- `show-sql=false` - Don't log SQL queries (production)
- `flyway.enabled=false` - Database migrations disabled

### Production Configuration (EC2)

**File:** `EC2:~/tutornation-backend/application.properties`

```properties
spring.application.name=TutorNation_Backend
spring.docker.compose.enabled=false

# H2 In-Memory Database Configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 Console (optional - for debugging)
spring.h2.console.enabled=true

# Flyway Database Migration
spring.flyway.enabled=false

# JPA/Hibernate Settings
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=false

# Batch Settings
spring.batch.job.enabled=false

# AI Configuration
spring.autoconfigure.exclude=org.springframework.ai.model.anthropic.autoconfigure.AnthropicChatAutoConfiguration

# Server port
server.port=8080
```

**⚠️ WARNING:** Currently using H2 in-memory database!
- Data is lost on every restart
- Only suitable for development/testing
- Should be migrated to PostgreSQL for production

### Environment Variables (set-env.sh)

**File:** `set-env.sh` (NOT currently used in deployment)

```bash
#!/bin/bash
export DB_URL="jdbc:postgresql://db.miicefbuykhjafjkhxgg.supabase.co:5432/postgres"
export DB_USERNAME="postgres"
export DB_PASSWORD="NpkPv5Cz9VrEXz2C"
```

This file contains Supabase credentials but is NOT being used in the current deployment.

---

## Troubleshooting

### Check Application Status

```bash
# Check if application is running
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'ps aux | grep java'

# Expected output:
# ec2-user   61978  0.2 20.3 3003820 191436 ?  Sl  Mar08  0:41 java -jar TutorNation_Backend-0.0.1-SNAPSHOT.jar
```

### View Application Logs

```bash
# View last 50 lines
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'tail -50 ~/tutornation-backend/app.log'

# Follow logs in real-time
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'tail -f ~/tutornation-backend/app.log'
```

### Test API Endpoints

```bash
# Test student endpoint
curl http://18.118.212.178:8080/api/students/hello

# Test tutor endpoint
curl http://18.118.212.178:8080/api/tutors/hello

# Test parent endpoint
curl http://18.118.212.178:8080/api/parents/hello
```

### Common Issues

#### 1. Application Won't Start

**Symptom:** No process running, check logs for errors

**Possible Causes:**
- Port 8080 already in use
- Database connection failed
- Configuration error
- Insufficient memory

**Solution:**
```bash
# Check what's using port 8080
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'netstat -tlnp | grep 8080'

# Check memory
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'free -h'

# View full logs
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'cat ~/tutornation-backend/app.log'
```

#### 2. Can't Connect from iOS App

**Possible Causes:**
- EC2 security group blocking port 8080
- Application crashed
- Wrong IP address in iOS app

**Solution:**
```bash
# Test from your machine
curl http://18.118.212.178:8080/api/students/hello

# Check EC2 security group in AWS Console:
# - Go to EC2 Dashboard
# - Select instance
# - Check "Security" tab
# - Ensure port 8080 is open to 0.0.0.0/0 (or your IP)
```

#### 3. Database Connection Errors

**Symptom:** Logs show "Connection refused" or "Unknown host"

**For H2 (current):** Shouldn't happen as H2 is in-memory

**For PostgreSQL (future):**
```bash
# Test database connection from EC2
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178

# Install PostgreSQL client
sudo dnf install postgresql15

# Test connection
psql -h db.miicefbuykhjafjkhxgg.supabase.co -U postgres -d postgres

# Check if EC2 can reach database
ping db.miicefbuykhjafjkhxgg.supabase.co
telnet db.miicefbuykhjafjkhxgg.supabase.co 5432
```

---

## Migration to Production Database

### Current State vs. Desired State

**Current (Development):**
```
EC2 Instance
    └── H2 In-Memory Database
        └── Data lost on restart
        └── No persistence
        └── Suitable for testing only
```

**Desired (Production):**
```
EC2 Instance ──────► PostgreSQL Database
    │                    (Supabase or AWS RDS)
    │                    └── Data persisted
    │                    └── Automatic backups
    │                    └── High availability
    └── API endpoints
```

### Migration Steps

#### Option 1: Use Existing Supabase Database

**1. Update EC2 application.properties:**

SSH to EC2 and edit the file:
```bash
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178

cd ~/tutornation-backend
nano application.properties
```

Replace H2 configuration with:
```properties
spring.application.name=TutorNation_Backend
spring.docker.compose.enabled=false

# PostgreSQL Database Configuration (Supabase)
spring.datasource.url=jdbc:postgresql://db.miicefbuykhjafjkhxgg.supabase.co:5432/postgres
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=postgres
spring.datasource.password=NpkPv5Cz9VrEXz2C

# Connection Pool Settings
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000

# Flyway Database Migration (optional - for schema versioning)
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true
spring.flyway.locations=classpath:db/migration

# JPA/Hibernate Settings
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true

# Batch Settings
spring.batch.job.enabled=false

# AI Configuration
spring.autoconfigure.exclude=org.springframework.ai.model.anthropic.autoconfigure.AnthropicChatAutoConfiguration

# Server port
server.port=8080
```

**2. Restart application:**
```bash
cd ~/tutornation-backend
./stop.sh
./start.sh
```

**3. Verify connection:**
```bash
tail -f app.log
# Look for: "HikariPool-1 - Added connection conn0: url=jdbc:postgresql://..."
```

#### Option 2: Create AWS RDS PostgreSQL Database

**1. Create RDS Instance (AWS Console):**

Navigate to: AWS Console → RDS → Create database

- **Engine:** PostgreSQL (latest version)
- **Template:** Free tier (for testing) or Production
- **DB Instance:** db.t3.micro (free tier) or larger
- **Settings:**
  - DB instance identifier: `tutornation-db`
  - Master username: `tutornadmin`
  - Master password: (create strong password)
- **Storage:** 20 GB GP3 (expandable)
- **Connectivity:**
  - VPC: Same as EC2 instance
  - Public access: No (more secure)
  - VPC Security group: Create new or use existing
  - Availability zone: us-east-2a (same as EC2)
- **Database authentication:** Password authentication
- **Additional configuration:**
  - Initial database name: `tutornation`
  - Automated backups: Enabled (7-35 days retention)
  - Encryption: Enabled

**2. Configure Security Group:**

RDS Security Group Inbound Rules:
```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source: EC2 instance security group (sg-xxxxx)
Description: Allow EC2 to connect to RDS
```

**3. Get RDS Endpoint:**

After creation (5-10 minutes):
- Go to RDS → Databases → tutornation-db
- Copy endpoint: `tutornation-db.xxxxx.us-east-2.rds.amazonaws.com`

**4. Update application.properties on EC2:**
```properties
spring.datasource.url=jdbc:postgresql://tutornation-db.xxxxx.us-east-2.rds.amazonaws.com:5432/tutornation
spring.datasource.username=tutornadmin
spring.datasource.password=YOUR_PASSWORD
```

**5. Restart and verify:**
```bash
cd ~/tutornation-backend
./stop.sh
./start.sh
tail -f app.log
```

### Database Schema Management

#### Option 1: Let JPA Create Tables (Quick Start)

Set in application.properties:
```properties
spring.jpa.hibernate.ddl-auto=update
```

This automatically creates/updates tables based on your @Entity classes.

**⚠️ Warning:** Not recommended for production as it can cause data loss.

#### Option 2: Use Flyway Migrations (Recommended)

**1. Create migration files:**

Create: `src/main/resources/db/migration/V1__Initial_schema.sql`
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tutoring_sessions (
    id BIGSERIAL PRIMARY KEY,
    tutor_id BIGINT REFERENCES users(id),
    student_id BIGINT REFERENCES users(id),
    subject VARCHAR(255) NOT NULL,
    scheduled_at TIMESTAMP NOT NULL,
    duration_minutes INT NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Add more tables as needed
```

**2. Enable Flyway in application.properties:**
```properties
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true
spring.flyway.locations=classpath:db/migration
```

**3. Deploy:**
```bash
./deploy.sh
```

Flyway will automatically:
1. Create `flyway_schema_history` table
2. Run V1__Initial_schema.sql
3. Track which migrations have been applied
4. Run only new migrations on subsequent deployments

---

## API Endpoints

### Current Endpoints

**Students API:**
```
GET http://18.118.212.178:8080/api/students
GET http://18.118.212.178:8080/api/students/hello
```

**Tutors API:**
```
GET http://18.118.212.178:8080/api/tutors
GET http://18.118.212.178:8080/api/tutors/hello
```

**Parents API:**
```
GET http://18.118.212.178:8080/api/parents
GET http://18.118.212.178:8080/api/parents/hello
```

### For iOS App

**Base URL Configuration:**
```swift
// In your iOS app NetworkManager or APIClient
let baseURL = "http://18.118.212.178:8080"

// Example usage
func fetchStudents() async throws -> [Student] {
    let url = URL(string: "\(baseURL)/api/students")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Student].self, from: data)
}
```

**⚠️ Note:** Using HTTP (not HTTPS). For production:
1. Get a domain name (e.g., api.tutornation.com)
2. Set up SSL certificate (AWS Certificate Manager + Load Balancer)
3. Use HTTPS for secure communication

---

## Security Considerations

### Current Security Issues

1. **No HTTPS** - Traffic not encrypted
2. **Credentials in code** - set-env.sh contains passwords
3. **H2 Console enabled** - Security risk if accessible
4. **No authentication** - API endpoints are public
5. **Port 8080 open to world** - Should restrict to specific IPs

### Recommended Improvements

**1. Use Environment Variables:**

Instead of application.properties with hardcoded credentials:

```bash
# On EC2, create /etc/environment.d/tutornation.conf
DB_URL=jdbc:postgresql://...
DB_USERNAME=postgres
DB_PASSWORD=NpkPv5Cz9VrEXz2C

# Or use AWS Systems Manager Parameter Store
aws ssm put-parameter --name /tutornation/db/password \
    --value "NpkPv5Cz9VrEXz2C" --type SecureString
```

Update start.sh:
```bash
export DB_URL="..."
export DB_USERNAME="postgres"
export DB_PASSWORD="..."
java -jar app.jar
```

**2. Add API Authentication:**

Implement JWT tokens or Spring Security:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // Configure authentication
}
```

**3. Use HTTPS:**

Set up Application Load Balancer with SSL certificate:
```
iOS Client → HTTPS (443) → ALB → HTTP (8080) → EC2
```

**4. Restrict Security Groups:**

Instead of 0.0.0.0/0, allow only:
- Your office IP
- iOS app IP ranges (if static)
- Or use VPN/bastion host

---

## Monitoring & Maintenance

### Log Rotation

Application logs will grow indefinitely. Set up log rotation:

```bash
# On EC2, create /etc/logrotate.d/tutornation
/home/ec2-user/tutornation-backend/app.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 644 ec2-user ec2-user
}
```

### Health Check Endpoint

Add to Spring Boot application:
```java
@RestController
public class HealthController {
    @GetMapping("/health")
    public Map<String, String> health() {
        return Map.of(
            "status", "UP",
            "timestamp", Instant.now().toString()
        );
    }
}
```

Test:
```bash
curl http://18.118.212.178:8080/health
```

### Automated Monitoring

Use AWS CloudWatch:
1. Install CloudWatch agent on EC2
2. Monitor CPU, memory, disk usage
3. Set up alarms for high resource usage
4. Send alerts to email/SNS

---

## Cost Considerations

### Current Costs (Estimated Monthly)

**EC2 Instance:**
- Type: t2.micro (free tier) or t3.small (~$15/month)
- Data transfer: Minimal (~$1/month)

**Database:**
- H2: $0 (in-memory, included with EC2)
- Supabase: Free tier (500MB) or paid plan
- RDS PostgreSQL: Free tier (db.t3.micro) or ~$15-50/month

**Total:** $0-65/month depending on configuration

### Cost Optimization

1. Use EC2 Reserved Instances (save 30-70%)
2. Use RDS free tier while developing
3. Stop EC2 when not needed (dev/test)
4. Use CloudWatch to monitor unused resources

---

## Backup Strategy

### Current State
**⚠️ NO BACKUPS** - H2 in-memory database loses all data on restart

### Recommended Backup Strategy

**With RDS:**
- Automated daily backups (free for 7 days)
- Manual snapshots before major changes
- Point-in-time recovery (up to 35 days)

**With Supabase:**
- Automatic backups included
- Download backups via Supabase dashboard

**Application-Level Backups:**
```bash
# Export data to JSON
curl http://18.118.212.178:8080/api/export > backup.json

# Or use pg_dump for PostgreSQL
pg_dump -h HOST -U USER -d DATABASE > backup.sql
```

---

## Future Improvements

1. **CI/CD Pipeline:**
   - GitHub Actions to automatically build and deploy on push
   - Automated testing before deployment

2. **Docker Containerization:**
   - Package app as Docker image
   - Deploy to ECS or EKS for better scalability

3. **Load Balancer:**
   - Add Application Load Balancer
   - Enable auto-scaling with multiple EC2 instances

4. **CDN:**
   - Use CloudFront for static assets
   - Reduce latency for global users

5. **Database Optimization:**
   - Add read replicas for better performance
   - Implement caching (Redis)

---

## Quick Reference

### Deployment Command
```bash
./deploy.sh
```

### View Logs
```bash
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'tail -f ~/tutornation-backend/app.log'
```

### Test Application
```bash
curl http://18.118.212.178:8080/api/students/hello
```

### Check Application Status
```bash
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'ps aux | grep java'
```

### Manual Restart
```bash
ssh -i /Users/minseok/tutornation-prod-us-east-2-keypair.pem \
    ec2-user@18.118.212.178 'cd ~/tutornation-backend && ./stop.sh && ./start.sh'
```

---

## Support

For issues or questions:
1. Check application logs first
2. Review this deployment guide
3. Check AWS EC2 and RDS console for infrastructure issues
4. Review Spring Boot documentation: https://spring.io/projects/spring-boot

---

**Last Updated:** March 8, 2026
**Version:** 1.0
**Maintained By:** TutorNation Development Team
