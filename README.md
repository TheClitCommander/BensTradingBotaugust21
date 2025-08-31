# TradingBenBot - Complete Trading System

This repository contains a complete trading system with backend, API, and frontend components.

## System Architecture

The system consists of three main components:

1. **Backend (trading_bot)**: Python-based trading engine, risk management, execution, and market context analysis
2. **API Server (live-api)**: Node.js Express server that provides API endpoints and WebSocket communication
3. **Frontend (new-trading-dashboard)**: React-based dashboard for visualization and interaction

## Backend Components

### API Module
- REST API endpoints for trading operations, market data, and system status
- WebSocket integration for real-time updates

### Engine Module
- Decision engine for policy-driven opportunity processing
- Asset selector for choosing optimal trading instruments
- Regime manager for adapting to market conditions

### Execution Module
- Order execution with bracket orders (entry + OCO stop/target)
- Time-In-Force (TIF) expiry management
- Execution watchdog for safety cancels
- Dynamic exit management (breakeven + ATR trailing stops)

### Risk Module
- Enhanced position sizing
- Heat risk gate for portfolio risk management
- Correlation-aware heat caps
- Drawdown protection

### Models Module
- Venue cost model for learning and predicting fees and slippage
- Calibration and meta-labeling for improved signal precision
- News alpha model for event-driven trading

### Market Context Module
- News features extraction
- Novelty detection
- Source reliability tracking

## API Server

The API server (live-api) provides:

- REST API endpoints for frontend communication
- WebSocket broadcasting for real-time updates
- Proxy to Python backend
- Fallback mechanisms for improved reliability

Recent fixes to the API server:
- Fixed handling of nested limit parameters in /api/bars
- Added proper /api/paper/orders/open endpoint
- Added proxying of key endpoints to FastAPI backend
- Added explicit control over synthetic data with ENABLE_STUBS flag

## Frontend Dashboard

The React-based dashboard includes:

### Components
- DataIngestionCard: Pipeline stages and events visualization
- OpenOrdersPanel: Display and management of open orders
- DecisionsTable: Trade decisions with detailed information
- RiskMeter: Portfolio risk visualization
- SafetyPill: System safety status indicator

### Styles
- Responsive utilities for fluid layouts
- Table styling for data presentation

### Utils
- Decision narrative generator for human-readable explanations

## Getting Started

### Setup

1. Install dependencies:
   ```
   cd live-api
   npm install http-proxy-middleware
   ```

### Running the System (Explicit & Honest Mode)

1. Start the FastAPI backend (serves the "truth": risk, decisions, portfolio, etc.):
   ```
   # from repo root
   uvicorn trading_bot.api.app:app --reload --port 8000
   ```

2. Start the Node API server:
   ```
   cd live-api
   
   # Option 1: With stubs enabled (for development without Python backend)
   ENABLE_STUBS=true PY_API=http://localhost:8000 node server.js
   
   # Option 2: Strict mode (no fake data, requires Python backend)
   ENABLE_STUBS=false PY_API=http://localhost:8000 node server.js
   ```

3. Start the frontend:
   ```
   cd new-trading-dashboard
   npm run dev
   ```

## Testing the System

1. Test API endpoints:
   ```
   curl -s http://localhost:4000/api/health | jq
   curl -s http://localhost:4000/api/data/status | jq
   curl -s -X POST http://localhost:4000/api/dev/ingest-test | jq
   curl -s "http://localhost:4000/api/ingestion/events?limit=1" | jq
   curl -s "http://localhost:4000/api/quotes?symbols=SPY,QQQ" | jq
   curl -s "http://localhost:4000/api/bars?symbol=SPY&limit=5" | jq
   curl -s http://localhost:4000/api/risk/summary | jq
   curl -s http://localhost:4000/api/decisions/recent | jq
   ```

2. Test creating a bracket order:
   ```
   curl -s -X POST http://localhost:4000/api/paper/orders \
    -H "Content-Type: application/json" \
    -d '{
      "entry": { "symbol":"SPY","side":"BUY","qty":1,"limit_price":450,"tif_seconds":90,"role":"entry","client_order_id":"dev-entry-1" },
      "oco": [
        { "symbol":"SPY","side":"SELL","qty":1,"stop_price":440,"role":"stop" },
        { "symbol":"SPY","side":"SELL","qty":1,"limit_price":460,"role":"target" }
      ],
      "oco_group_id": "dev-oco-1",
      "venue":"paper",
      "mid_submit": 450.10
    }' | jq
   ```

3. Verify the order in the open orders list:
   ```
   curl -s http://localhost:4000/api/paper/orders/open | jq
   ```

## Recent Fixes

1. **Fixed 404 for Open Orders**:
   - Added proper implementation of GET /api/paper/orders/open
   - Added aliases for compatibility with different UI versions

2. **Fixed 500/502 for /api/risk/, /api/decisions/, etc.**:
   - Added proper proxying to FastAPI backend
   - Implemented clear error responses when backend is unavailable

3. **Fixed 502 for /api/bars**:
   - Fixed handling of nested limit parameters (limit[limit]=30)
   - Added readNumeric helper to safely parse query parameters

4. **Improved Synthetic Data Handling**:
   - Added ENABLE_STUBS flag to control when synthetic data is returned
   - Made it explicit when synthetic data is being used
   - Returns proper error codes when real data sources are unavailable

5. **UI Responsiveness**:
   - Added responsive CSS utilities for better layout
   - Improved card components for fluid sizing
   - Enhanced table components for better mobile display
