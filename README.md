# Network Intrusion Detection System

A comprehensive real-time network intrusion detection system with ML-powered threat analysis, live dashboards, and automated alerting.

## Architecture

\`\`\`
┌─────────────────┐
│  Next.js        │
│  Frontend       │  (Dashboard, Logs, Alerts)
└────────┬────────┘
         │
┌────────▼────────┐
│  Next.js API    │  (/api/ingest, /api/stats, /api/alerts)
│  Routes         │
└────────┬────────┘
         │
    ┌────┴────┐
    │          │
┌───▼──┐   ┌──▼────┐
│      │   │       │
│FastAPI   Supabase│
│ML Model  Database│
│Service   + Auth  │
│          + RLS   │
└──────┘   └───────┘
\`\`\`

## Features

- **Real-time Traffic Monitoring**: Ingest network traffic and classify as normal/attack
- **ML-Powered Detection**: Random Forest model for accurate intrusion classification
- **Live Dashboard**: Real-time statistics, charts, and attack distribution
- **Alert System**: Email, SMS, and UI notifications for detected threats
- **Persistent Storage**: All logs and alerts stored in Supabase with RLS
- **Role-Based Access**: Admin, analyst, and viewer roles
- **Supabase Realtime**: Live updates via WebSockets

## Setup Instructions

### Prerequisites
- Node.js 18+
- Python 3.11+
- Docker & Docker Compose (optional)
- Supabase project (create at supabase.com)

### 1. Environment Setup

Copy `.env.example` to `.env.local` and update with your values:

\`\`\`bash
cp .env.example .env.local
\`\`\`

Update the environment variables:
- `NEXT_PUBLIC_SUPABASE_URL`: Your Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Your Supabase anon key
- `SUPABASE_SERVICE_ROLE_KEY`: Your Supabase service role key
- `DATABASE_URL`: PostgreSQL connection string (use Supabase's connection string)

### 2. Database Setup

Run the SQL migrations in Supabase:

\`\`\`bash
# Option 1: Run directly in Supabase SQL editor
# Copy contents of scripts/01-init-schema.sql
# Paste in Supabase SQL editor and execute

# Option 2: Using psql CLI
psql $DATABASE_URL -f scripts/01-init-schema.sql
psql $DATABASE_URL -f scripts/02-init-realtime.sql
\`\`\`

### 3. Generate Sample Model

\`\`\`bash
python scripts/generate_sample_model.py
\`\`\`

This creates `ml_service/model.pkl` with a pre-trained model for testing.

### 4. Install Dependencies

\`\`\`bash
# Frontend/Backend
npm install

# ML Service
cd ml_service
pip install -r requirements.txt
cd ..
\`\`\`

### 5. Run Services

**Option A: Local Development**

Terminal 1 - Start ML microservice:
\`\`\`bash
cd ml_service
python -m uvicorn app:app --host 0.0.0.0 --port 5000 --reload
\`\`\`

Terminal 2 - Start Next.js:
\`\`\`bash
npm run dev
\`\`\`

Terminal 3 - Simulate traffic:
\`\`\`bash
python scripts/simulate_traffic.py
\`\`\`

**Option B: Docker Compose**

\`\`\`bash
docker-compose up --build
\`\`\`

Then in another terminal:
\`\`\`bash
python scripts/simulate_traffic.py
\`\`\`

### 6. Access the Application

- **Frontend**: http://localhost:3000
- **Dashboard**: http://localhost:3000/dashboard
- **ML API**: http://localhost:5000
- **API Docs**: http://localhost:5000/docs

## API Endpoints

### Traffic Ingest
\`\`\`bash
POST /api/ingest
Content-Type: application/json

{
  "duration": 1.5,
  "protocol_type": "tcp",
  "src_bytes": 100,
  "dst_bytes": 200,
  "src_port": 1024,
  "dst_port": 80,
  "flag": "S0",
  "land": 0,
  "wrong_fragment": 0,
  "urgent": 0
}
\`\`\`

### Get Statistics
\`\`\`bash
GET /api/stats?hours=24
\`\`\`

### Get Alerts
\`\`\`bash
GET /api/alerts?limit=50
\`\`\`

### Update Alert
\`\`\`bash
PATCH /api/alerts
Content-Type: application/json

{
  "alertId": "uuid",
  "acknowledged": true
}
\`\`\`

### Get/Update Settings
\`\`\`bash
GET /api/settings
PUT /api/settings
\`\`\`

## Deployment

### Deploy to Vercel

\`\`\`bash
# Frontend deployment
vercel deploy
\`\`\`

### Deploy ML Service

Deploy to Render, Railway, or Heroku:

\`\`\`bash
# Railway example
railway link
railway deploy
\`\`\`

### Environment Variables

Set these in your deployment platform:
- All variables from `.env.example`
- `FASTAPI_URL`: URL of deployed FastAPI service

## Database Schema

### traffic_logs
- Stores all network traffic predictions
- Real-time subscription support
- Indexed on timestamp, prediction, severity

### alerts
- Triggered alerts from detected attacks
- Acknowledgment tracking
- Severity and rule information

### alert_rules
- Custom alert conditions
- Enable/disable toggle
- Action types (email, SMS, UI)

### alert_config
- User-specific alert preferences
- Email and SMS settings
- Minimum severity threshold

### system_settings
- Global configuration
- Alert thresholds
- Service credentials

## Performance Optimization

- Index on `timestamp DESC` for fast historical queries
- Materialized views for hourly statistics
- Real-time subscriptions for live updates
- Connection pooling via Supabase

## Troubleshooting

**ML Service Connection Error**
- Ensure FastAPI is running on port 5000
- Check `FASTAPI_URL` environment variable

**No Data in Dashboard**
- Run `scripts/simulate_traffic.py` to generate sample traffic
- Check Supabase connection

**Realtime Not Working**
- Verify Realtime is enabled in Supabase
- Check RLS policies are correct

## Future Enhancements

- [ ] PCAP file upload support
- [ ] Advanced rule engine
- [ ] Threat intelligence integration
- [ ] Automated response playbooks
- [ ] Multi-model ensemble detection
- [ ] GraphQL API

## License

MIT
