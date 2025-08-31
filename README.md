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

Two versions are included:
- `server.js`: Original version
- `server-fixed.js`: Enhanced version with improved fallbacks and error handling

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

1. Start the backend:
   ```
   cd trading_bot
   python main.py
   ```

2. Start the API server:
   ```
   cd live-api
   export ALLOW_SYNTHETIC_FALLBACK=true
   node server-fixed.js
   ```

3. Start the frontend:
   ```
   cd new-trading-dashboard
   npm install
   npm run dev
   ```

## Testing the System

1. Test API endpoints:
   ```
   curl -s http://localhost:4000/health | jq
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

## Recent Improvements

1. **Enhanced API Server**:
   - Added proper ingestion metrics and events endpoints
   - Added quotes and bars fallbacks that work without providers
   - Added risk summary and heat stubs
   - Added decisions stubs and a seed-decision endpoint
   - Fixed the open orders endpoint

2. **UI Responsiveness**:
   - Added responsive CSS utilities for better layout
   - Improved card components for fluid sizing
   - Enhanced table components for better mobile display

3. **New Components**:
   - DataIngestionCard: Pipeline stages and events visualization
   - OpenOrdersPanel: Display and management of open orders
   - Decision Narrative: Human-readable explanations for trade decisions
