# DBDock

Enterprise-grade PostgreSQL backup and restore. Beautiful CLI with real-time progress tracking.

[![npm version](https://img.shields.io/npm/v/dbdock.svg)](https://www.npmjs.com/package/dbdock)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org)
[![Documentation](https://img.shields.io/badge/docs-dbdock.mintlify.app-blue)](https://dbdock.mintlify.app)

📚 **[Full Documentation](https://dbdock.mintlify.app)** | 💬 **[Discussions](https://github.com/naheemolaide/dbdock-support/discussions)** | 🐛 **[Issues](https://github.com/naheemolaide/dbdock-support/issues)**

## Quick Start

```bash
npx dbdock init      # Interactive setup
npx dbdock backup    # Create backup
npx dbdock restore   # Restore backup
```

## Features

- **Beautiful CLI** - Real-time progress bars, speed tracking, smart filtering
- **Multiple Storage** - Local, AWS S3, Cloudflare R2, Cloudinary
- **Security** - AES-256 encryption, Brotli compression
- **Retention Policies** - Automatic cleanup by count/age with safety nets
- **Smart UX** - Intelligent filtering for 100+ backups, clear error messages
- **Email Alerts** - SMTP notifications with custom templates
- **TypeScript Native** - Full type safety for programmatic usage
- **Automation** - Cron schedules, auto-cleanup after backups

## Installation

**Global Installation (Recommended):**

```bash
npm install -g dbdock-cli

dbdock init      # Use directly
dbdock backup
dbdock status
```

**Or use with npx (No installation needed):**

```bash
npx dbdock init
npx dbdock backup
npx dbdock status
```

## CLI Commands

### `dbdock init`

Interactive setup wizard that creates `dbdock.config.json` with:

- Database connection (host, port, credentials)
- Storage provider (Local, S3, R2, Cloudinary)
- Encryption/compression settings
- Email alerts (optional)

Auto-adds config to `.gitignore` to protect credentials.

### `npx dbdock backup`

Creates database backup with real-time progress tracking:

```
████████████████████ | 100% | 45.23/100 MB | Speed: 12.50 MB/s | ETA: 0s | Uploading to S3
✔ Backup completed successfully
```

**Options:**

```bash
npx dbdock backup --encrypt --compress --compression-level 9
```

- `--encrypt` / `--no-encrypt` - Toggle encryption
- `--compress` / `--no-compress` - Toggle compression
- `--encryption-key <key>` - 64-char hex key (must be exactly 64 hexadecimal characters)
- `--compression-level <1-11>` - Compression level (default: 6)

**Generate encryption key:**

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**Backup Formats:**

- `custom` (default) - PostgreSQL custom binary format (.sql)
- `plain` - Plain SQL text format (.sql)
- `directory` - Directory format (.dir)
- `tar` - Tar archive format (.tar)

### `npx dbdock restore`

Interactive restore with smart filtering and multi-step progress:

```
Progress:
────────────────────────────────────────────────────────
  ✔ Downloading backup
  ✔ Decrypting data
  ✔ Decompressing data
  ⟳ Restoring to database...
────────────────────────────────────────────────────────
✔ All steps completed in 8.42s
```

**Smart filtering** (auto-enabled for 20+ backups):

- Show recent (last 10)
- Date range (24h, 7d, 30d, 90d, custom)
- Search by keyword/ID

Shows database stats and requires confirmation before restore.

### `npx dbdock list`

View backups with smart filtering:

```bash
npx dbdock list                  # All backups
npx dbdock list --recent 10      # Last 10
npx dbdock list --search keyword # Search
npx dbdock list --days 7         # Last 7 days
```

**Auto-filtering** for 50+ backups with interactive prompts.

### `npx dbdock delete`

Delete backups interactively or by key:

```bash
npx dbdock delete              # Interactive
npx dbdock delete --key <id>   # Specific backup
npx dbdock delete --all        # All (with confirmation)
```

### `npx dbdock cleanup`

Clean up old backups based on retention policy:

```bash
npx dbdock cleanup              # Interactive with preview
npx dbdock cleanup --dry-run    # Preview only
npx dbdock cleanup --force      # Skip confirmation
```

Shows detailed preview of what will be deleted and space to reclaim.

### `dbdock status`

Quick view of all schedules and service status:

```bash
dbdock status
```

**Output:**

```
📅 Scheduled Backups:

┌─────┬──────────────┬─────────────────┬──────────┐
│  #  │ Name         │ Cron Expression │ Status   │
├─────┼──────────────┼─────────────────┼──────────┤
│   1 │ daily        │ 0 * * * *       │ ✓ Active │
│   2 │ weekly       │ 0 0 * * 0       │ ✗ Paused │
└─────┴──────────────┴─────────────────┴──────────┘

Total: 2 schedule(s) - 1 active, 1 paused

🚀 Service Status:

🟢 Running (PM2)
  PID: 12345
  Uptime: 2d 5h
  Memory: 45.23 MB
```

### `dbdock test`

Validates database, storage, and email configuration.

### `dbdock schedule`

Manage backup schedules in configuration:

```bash
dbdock schedule
```

**Features:**

- View current schedules with status
- Add new schedule with cron expression presets
- Remove or toggle (enable/disable) schedules
- Saves to `dbdock.config.json`

**Schedule Presets:**

- Every hour: `0 * * * *`
- Every day at midnight: `0 0 * * *`
- Every day at 2 AM: `0 2 * * *`
- Every week (Sunday): `0 0 * * 0`
- Every month (1st): `0 0 1 * *`
- Custom cron expression

**⚠️ Important:** Schedules only execute when DBDock is integrated into your NestJS application (see Programmatic Usage below). The CLI is for configuration only.

## Configuration

After running `npx dbdock init`, a `dbdock.config.json` file is created:

```json
{
  "database": {
    "type": "postgres",
    "host": "localhost",
    "port": 5432,
    "username": "postgres",
    "password": "your-password",
    "database": "myapp"
  },
  "storage": {
    "provider": "s3",
    "s3": {
      "bucket": "my-backups",
      "region": "us-east-1",
      "accessKeyId": "YOUR_ACCESS_KEY",
      "secretAccessKey": "YOUR_SECRET_KEY"
    }
  },
  "backup": {
    "format": "custom",
    "compression": {
      "enabled": true,
      "level": 6
    },
    "encryption": {
      "enabled": true,
      "key": "64-character-hex-key-generated-with-openssl-or-node-crypto"
    },
    "retention": {
      "enabled": true,
      "maxBackups": 100,
      "maxAgeDays": 30,
      "minBackups": 5,
      "runAfterBackup": true
    }
  },
  "alerts": {
    "email": {
      "enabled": true,
      "smtp": {
        "host": "smtp.gmail.com",
        "port": 587,
        "secure": false,
        "auth": {
          "user": "your-email@gmail.com",
          "pass": "your-app-password"
        }
      },
      "from": "backups@yourapp.com",
      "to": ["admin@yourapp.com"]
    }
  }
}
```

### Storage Providers

**Local:**

```json
{ "storage": { "provider": "local", "local": { "path": "./backups" } } }
```

**AWS S3:**

```json
{
  "storage": {
    "provider": "s3",
    "s3": {
      "bucket": "my-backups",
      "region": "us-east-1",
      "accessKeyId": "YOUR_KEY",
      "secretAccessKey": "YOUR_SECRET"
    }
  }
}
```

Required permissions: `s3:PutObject`, `s3:GetObject`, `s3:ListBucket`, `s3:DeleteObject`

**Cloudflare R2:**

```json
{
  "storage": {
    "provider": "r2",
    "s3": {
      "bucket": "my-backups",
      "region": "auto",
      "endpoint": "https://ACCOUNT_ID.r2.cloudflarestorage.com",
      "accessKeyId": "YOUR_KEY",
      "secretAccessKey": "YOUR_SECRET"
    }
  }
}
```

**Cloudinary:**

```json
{
  "storage": {
    "provider": "cloudinary",
    "cloudinary": {
      "cloudName": "your-cloud",
      "apiKey": "YOUR_KEY",
      "apiSecret": "YOUR_SECRET"
    }
  }
}
```

All cloud backups stored in `dbdock_backups/` folder with format: `backup-YYYY-MM-DD-HH-MM-SS-BACKUPID.sql`

### Retention Policy

Automatic cleanup to prevent storage bloat from frequent backups:

```json
{
  "backup": {
    "retention": {
      "enabled": true,
      "maxBackups": 100,
      "maxAgeDays": 30,
      "minBackups": 5,
      "runAfterBackup": true
    }
  }
}
```

**How it works:**

- Keeps most recent `minBackups` (safety net, never deleted)
- Deletes backups exceeding `maxBackups` limit (oldest first)
- Deletes backups older than `maxAgeDays` (respecting minBackups)
- Runs automatically after each backup (if `runAfterBackup: true`)
- Manual cleanup: `npx dbdock cleanup`

**Safety features:**

- Always preserves `minBackups` most recent backups
- Shows preview before deletion
- Detailed logging of what was deleted
- Error handling for failed deletions

## Programmatic Usage

### NestJS Integration

```typescript
import { Module } from '@nestjs/common';
import { DBDockModule } from 'dbdock';

@Module({
  imports: [
    DBDockModule.forRoot({
      database: {
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'postgres',
        password: process.env.DB_PASSWORD,
        database: 'myapp',
      },
      storage: {
        provider: 's3',
        s3: {
          bucket: 'my-backups',
          region: 'us-east-1',
          accessKeyId: process.env.AWS_ACCESS_KEY_ID,
          secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
        },
      },
      backup: {
        compression: { enabled: true, level: 6 },
        encryption: { enabled: true, key: process.env.ENCRYPTION_KEY },
        schedules: [
          { name: 'Daily Backup', cron: '0 2 * * *', enabled: true },
          { name: 'Weekly Full', cron: '0 0 * * 0', enabled: true },
        ],
      },
    }),
  ],
})
export class AppModule {}
```

### Creating and Restoring Backups

```typescript
import { BackupService } from 'dbdock';

@Injectable()
export class MyService {
  constructor(private backupService: BackupService) {}

  async createBackup() {
    const result = await this.backupService.createBackup({
      compress: true,
      encrypt: true,
    });

    console.log(`Backup created: ${result.metadata.id}`);
    return result;
  }

  async restore(backupId: string) {
    await this.backupService.restoreBackup(backupId);
  }
}
```

### Schedule Management API

```typescript
import { ScheduleManager } from 'dbdock';

const scheduleManager = new ScheduleManager();

scheduleManager.addSchedule({
  name: 'Hourly Backup',
  cron: '0 * * * *',
  enabled: true,
});

const schedules = scheduleManager.getSchedules();
console.log('All schedules:', schedules);

scheduleManager.updateSchedule('Hourly Backup', {
  cron: '0 */2 * * *',
});

scheduleManager.enableSchedule('Daily Backup');
scheduleManager.disableSchedule('Weekly Full');

scheduleManager.removeSchedule('Old Schedule');
```

### Complete Example with Schedules

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ScheduleManager } from 'dbdock';

async function bootstrap() {
  const scheduleManager = new ScheduleManager();

  scheduleManager.addSchedule({
    name: 'Daily Backup',
    cron: '0 2 * * *',
    enabled: true,
  });

  scheduleManager.addSchedule({
    name: 'Weekly Archive',
    cron: '0 0 * * 0',
    enabled: true,
  });

  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  console.log('Application started with scheduled backups');
}

bootstrap();
```

## Requirements

- Node.js 18 or higher
- PostgreSQL 12+
- PostgreSQL client tools (`pg_dump`, `pg_restore`, `psql`)

**Installing PostgreSQL client tools:**

```bash
# macOS
brew install postgresql

# Ubuntu/Debian
sudo apt-get install postgresql-client

# Windows
# Download from https://www.postgresql.org/download/windows/
```

## Troubleshooting

Run `npx dbdock test` to verify your configuration.

### Common Issues

**pg_dump not found:**

```bash
# macOS
brew install postgresql

# Ubuntu/Debian
sudo apt-get install postgresql-client
```

**Database connection errors:**

- Verify `host`, `port`, `username`, `password`, `database` in config
- Test connection: `psql -h HOST -p PORT -U USERNAME -d DATABASE`
- Check PostgreSQL server is running
- Verify network/firewall allows connection

**Storage errors:**

_AWS S3:_

- Verify credentials are correct
- Ensure IAM user has permissions: `s3:PutObject`, `s3:GetObject`, `s3:ListBucket`, `s3:DeleteObject`
- Check bucket name and region

_Cloudflare R2:_

- Verify API token is correct
- Check endpoint URL format: `https://ACCOUNT_ID.r2.cloudflarestorage.com`
- Ensure bucket exists and is accessible
- Verify R2 credentials have read/write permissions

_Cloudinary:_

- Verify cloud name, API key, and secret are correct
- Check your Cloudinary account is active
- Ensure API credentials have media library access

**Encryption key errors:**

```bash
# Generate a valid 64-character hex key
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Must be exactly 64 hexadecimal characters (0-9, a-f, A-F)
```

**R2 restore not working:**

- Ensure backups are in `dbdock_backups/` folder
- Verify backup files are named with `.sql` extension
- Check endpoint configuration matches R2 account ID

**No backups found:**

- Local: Check files exist in configured path
- S3/R2: Verify files are in `dbdock_backups/` folder
- Cloudinary: Check Media Library for `dbdock_backups` folder
- Ensure files match pattern: `backup-*.sql`

DBDock shows clear, actionable error messages for all issues with specific troubleshooting steps.

## Documentation

📚 **[Full Documentation](https://dbdock.mintlify.app)** - Comprehensive guides, API reference, and examples

## Support

- 💬 **[Discussions](https://github.com/naheemolaide/dbdock-support/discussions)** - Ask questions and share ideas
- 🐛 **[Issues](https://github.com/naheemolaide/dbdock-support/issues)** - Report bugs and request features

## Links

- 📦 **[npm Package](https://www.npmjs.com/package/dbdock)**
- 📖 **[Documentation](https://dbdock.mintlify.app)**
- 💻 **[GitHub Repository](https://github.com/naheemolaide/dbdock-support)**

## License

MIT
