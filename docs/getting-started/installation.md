---
layout: default
title: "Installation Guide"
description: "Step-by-step guide to install and set up Masetra Quality Management System"
---

# Installation Guide

Learn how to install and set up Masetra for your organization.

## 📋 System Requirements

### Server Requirements

- **Operating System:** Ubuntu 20.04+ or CentOS 8+
- **RAM:** 8GB minimum (16GB recommended)
- **Storage:** 50GB available disk space
- **Database:** PostgreSQL 13+
- **Web Server:** Nginx or Apache

### Client Requirements

- **Browser:** Latest Chrome, Firefox, Safari, or Edge
- **Mobile:** iOS 12+ or Android 8+
- **Internet:** Stable broadband connection

## 🚀 Installation Options

### Cloud Deployment (Recommended)

Masetra is available as a managed cloud service with:

- Automatic updates
- 99.9% uptime SLA
- 24/7 monitoring
- Professional support

Contact info@masetra.com for pricing.

### Self-Hosted Installation

#### Prerequisites

1. Install Docker and Docker Compose
2. Install PostgreSQL 13+
3. Configure domain and SSL certificates

#### Installation Steps

1. **Download Installation Package**

```bash
wget https://downloads.masetra.com/masetra-latest.tar.gz
tar -xzf masetra-latest.tar.gz
cd masetra
```

2. **Configure Environment**

```bash
cp .env.example .env
# Edit .env with your configuration
```

3. **Start Services**

```bash
docker-compose up -d
```

4. **Initialize Database**

```bash
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
```

5. **Access Application**

Open your browser to https://your-domain.com

🔧 Configuration
Environment Variables

```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=masetra
DB_USER=masetra_user
DB_PASSWORD=secure_password

# Email Configuration
EMAIL_HOST=smtp.your-email.com
EMAIL_PORT=587
EMAIL_USER=notifications@masetra.com
EMAIL_PASSWORD=secure_email_password

# Security Settings
SECRET_KEY=your_secret_key_here
DEBUG=False
ALLOWED_HOSTS=your-domain.com,localhost
```


🧪 Post-Installation Verification

1. Check Services Status

```bash
docker-compose ps
```

2. Verify Database Connection

```bash
docker-compose exec db psql -U masetra_user -d masetra
```

3. Test Web Interface
Navigate to your domain and verify login page loads.

🆘 Troubleshooting
Common Issues
Database Connection Failed

- Verify database credentials in .env
- Check if PostgreSQL service is running
- Ensure firewall allows database port

Application Not Loading

- Check Docker container logs: docker-compose logs web
- Verify domain configuration
- Confirm SSL certificates are valid

🔄 Updates
To update to the latest version:

```bash
docker-compose down
wget https://downloads.masetra.com/masetra-latest.tar.gz
tar -xzf masetra-latest.tar.gz
docker-compose up -d
```

📞 Support
For installation assistance, contact info@masetra.com or call +256-760-927180.

Last updated: {{ site.time | date: '%B %d, %Y' }}