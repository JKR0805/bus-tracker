# Bus Tracker

A real-time GPS-based bus tracking system with automatic stop detection, location clustering, and role-based user management. Built for school/college transportation management with driver and student interfaces.

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![Flask](https://img.shields.io/badge/flask-3.0.3-green.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 📋 Table of Contents

- [Features](#features)
- [System Architecture](#system-architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API Documentation](#api-documentation)
- [Database Schema](#database-schema)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## ✨ Features

### Core Functionality
- **Real-time GPS Tracking**: Driver location updates bus position automatically
- **Automatic Stop Detection**: Smart detection when bus arrives at stops (within 40 meters)
- **Daily Reset System**: Automatically resets bus to starting stop at midnight
- **Role-Based Access**: Separate interfaces for drivers and students
- **Session Management**: Secure, persistent user sessions
- **Interactive Maps**: Real-time Leaflet.js maps with stop markers

### Advanced Features
- **Driver Live Location**: Bus position updates in real-time from driver's GPS
- **Smart Proximity Detection**: Auto-detects when driver is near a stop
- **Fallback Manual Controls**: Arrived/Departed buttons work when GPS is unavailable (>30s)
- **Manual Reset Button**: Driver can manually reset bus to starting point
- **Map Center Control**: "Center on Bus" button for easy navigation
- **Toast Notifications**: User-friendly feedback messages
- **Responsive Design**: Mobile-first UI for on-the-go usage

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Driver UI  │  │  Student UI  │  │   Login UI   │      │
│  │  (driver.html)│  │(student.html)│  │ (login.html) │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         │                   │                   │            │
│         └───────────────────┴───────────────────┘            │
│                         │                                    │
│                  Leaflet.js Maps                             │
│                  Real-time AJAX                              │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                     Backend Layer (Flask)                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  app.py - Main Application                           │   │
│  │  • Authentication & Session Management               │   │
│  │  • Route Handlers                                    │   │
│  │  • Location Processing                               │   │
│  │  • API Endpoints                                     │   │
│  └──────────────────────────────────────────────────────┘   │
│                         │                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  db.py - Database Layer                              │   │
│  │  • Location Clustering                               │   │
│  │  • Stop Detection                                    │   │
│  │  • State Management                                  │   │
│  │  • Distance Calculations                             │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                   Data Layer (SQLite)                        │
│  • bus_state         - Current bus location & status        │
│  • stops             - Stop locations & sequences           │
│  • user_locations    - Real-time user GPS data              │
│  • confirmations     - Student arrival confirmations        │
│  • location_clusters - Aggregated location clusters         │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Prerequisites

- Python 3.8 or higher
- pip (Python package installer)
- Modern web browser with GPS support
- SQLite3 (included with Python)

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/bus-tracker-v3.git
cd bus-tracker-v3
```

### 2. Create Virtual Environment (Recommended)

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Initialize Database

The database will be automatically initialized on first run. To manually initialize:

```bash
python -c "from db import init_db; init_db()"
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the project root (optional):

```env
FLASK_SECRET_KEY=your-secret-key-here-change-in-production
FLASK_ENV=development
FLASK_DEBUG=True
```

### Default Users

The system comes with pre-configured test users (defined in `app.py`):

#### Driver Account
- **Username**: `driver1`
- **Password**: `driverpass123`
- **Bus ID**: `S1/A`

#### Student Accounts
- **Username**: `student1`, `student2`, `student3`
- **Password**: `studentpass123`

⚠️ **Important**: Change these credentials in production!

### Bus Configuration

Edit `app.py` to modify bus settings:

```python
BUS_ID = "S1/A"  # Default bus identifier
QUORUM = 1       # Number of confirmations needed to move bus
```

### Stop Configuration

Modify stops in `db.py`:

```python
SEED_STOPS = [
    ('Starting Point', 17.495643, 78.335691, 0),
    ('Stop A', 17.495255, 78.340605, 1),
    # Add more stops...
]
```

## 🎯 Usage

### Starting the Application

#### Development Mode

```bash
python app.py
```

The application will be available at `http://localhost:5000`

#### Production Mode with Gunicorn

```bash
gunicorn -w 4 -b 0.0.0.0:8000 wsgi:application
```

### User Workflows

#### For Drivers

1. **Login** at `/login` with driver credentials
2. **Share Location**: Click the location sharing button (red pin icon) to start GPS tracking
3. **Automatic Updates**: System auto-detects arrival at stops based on GPS proximity
4. **Monitor**: View current stop, next stop, and status updates
5. **Manual Override**: Use "DEPARTED" button only if GPS is unavailable (shows error if GPS is active)
6. **Reset Bus**: Click reset button to manually return bus to starting stop

**GPS Tracking Behavior:**
- When GPS is active (updated within last 30 seconds), manual buttons are disabled
- Bus automatically moves to detected stop when driver is within 40 meters
- "At Stop" status set automatically when near a stop
- "Departing" status set automatically when moving away from stops

#### For Students

1. **Login** at `/login` with student credentials
2. **Track Bus**: View real-time bus location on map updated by driver's GPS
3. **Navigate Map**: Freely pan and zoom the map
4. **Center on Bus**: Click the center button to quickly locate the bus
5. **Confirm Arrival**: Press "ARRIVED" when bus reaches your stop (optional)
6. **Location Sharing**: Button shows "Coming Soon" - feature reserved for future release

**Note:** Students can only view the bus location, not share their own location.

## 📡 API Documentation

### Authentication Endpoints

#### POST `/login`
Login user and create session.

**Request Body:**
```json
{
  "username": "driver1",
  "password": "driverpass123"
}
```

**Response:**
- Redirects to appropriate dashboard based on role

#### GET `/logout`
Logout user and clear session.

### Location Endpoints

#### POST `/location/share`
Share current GPS location (**driver only** - students receive 403 error).

**Request Body:**
```json
{
  "bus_id": "S1/A",
  "lat": 17.495643,
  "lon": 78.335691,
  "accuracy": 10.5
}
```

**Response:**
```json
{
  "bus_id": "S1/A",
  "stop_index": 2,
  "lat": 17.496050,
  "lon": 78.358307,
  "stop_name": "Stop B",
  "stop_id": 2,
  "status": "arrived",
  "accuracy": 10.5
}
```

**Behavior:**
- Updates bus location to driver's GPS coordinates
- Auto-detects proximity to stops (within 40 meters)
- Automatically changes stop when driver approaches a new stop
- Sets status to "arrived" when near a stop, "departing" otherwise

**Rate Limit:** 1 request per second per user

#### GET `/location/active?bus_id=S1/A`
Get active location sharers and clusters (**deprecated** - now only driver shares location).

**Response:**
```json
{
  "driver": {
    "last_update": "2025-10-17T10:30:45Z",
    "accuracy": 8.5
  },
  "active_users": 1,
  "last_update": "2025-10-17T10:30:45Z"
}
```

### Bus State Endpoints

#### GET `/bus/<bus_id>`
Get current bus state.

**Response:**
```json
{
  "bus_id": "S1/A",
  "stop_index": 2,
  "lat": 17.496050,
  "lon": 78.358307,
  "timestamp": "2025-10-17T10:30:45Z",
  "status": "departing",
  "stop_name": "Stop B",
  "stop_id": 2
}
```

#### GET `/stops`
Get all bus stops.

**Response:**
```json
[
  {
    "id": 1,
    "name": "Starting Point",
    "lat": 17.495643,
    "lon": 78.335691,
    "seq": 0
  }
]
```

### Driver Endpoints

#### POST `/driver/departed`
Mark bus as departed from current stop (**fallback only - disabled if GPS active**).

**Request Body:**
```json
{
  "bus_id": "S1/A"
}
```

**Response:** 
```json
{
  "error": "Using live GPS - manual control disabled",
  "message": "Bus position is being tracked via GPS"
}
```
*Returns 400 error if driver GPS was updated within last 30 seconds*

**Response (when GPS inactive):**
Updated bus state

#### POST `/driver/arrived`
Manually mark arrival at next stop (**fallback only - disabled if GPS active**).

**Request Body:**
```json
{
  "bus_id": "S1/A",
  "action": "arrived"
}
```

**Response:** Same as `/driver/departed` - requires GPS to be inactive

#### POST `/driver/reset`
Reset bus to starting stop (manual override for testing).

**Request Body:**
```json
{
  "bus_id": "S1/A"
}
```

**Response:**
```json
{
  "bus_id": "S1/A",
  "stop_index": 0,
  "lat": 17.495643,
  "lon": 78.335691,
  "stop_name": "Starting Point",
  "stop_id": 1,
  "status": null
}
```

### Student Endpoints

#### POST `/student/arrived`
Confirm arrival at current stop (student only).

**Request Body:**
```json
{
  "bus_id": "S1/A"
}
```

**Response:**
```json
{
  "confirmations": 2,
  "moved": false,
  "state": { /* bus state */ }
}
```

#### GET `/confirmations?bus_id=S1/A&stop_id=2`
Get confirmation count for a stop.

**Response:**
```json
{
  "bus_id": "S1/A",
  "stop_id": 2,
  "confirmations": 2
}
```

## 🗄️ Database Schema

### `stops`
Stores bus stop information.

| Column | Type    | Description           |
|--------|---------|-----------------------|
| id     | INTEGER | Primary key           |
| name   | TEXT    | Stop name             |
| lat    | REAL    | Latitude              |
| lon    | REAL    | Longitude             |
| seq    | INTEGER | Sequence/order number |

### `bus_state`
Tracks current bus location and status.

| Column               | Type    | Description                      |
|----------------------|---------|----------------------------------|
| bus_id               | TEXT    | Primary key, bus identifier      |
| stop_index           | INTEGER | Current stop sequence            |
| lat                  | REAL    | Current latitude                 |
| lon                  | REAL    | Current longitude                |
| timestamp            | TEXT    | Last update time (ISO 8601)      |
| status               | TEXT    | 'arrived', 'departing', etc.     |
| location_source      | TEXT    | 'driver', 'students', etc.       |
| location_accuracy    | REAL    | GPS accuracy in meters           |
| sample_size          | INTEGER | Number of GPS samples used       |
| last_arrival_time    | TEXT    | Last arrival timestamp           |
| last_departure_time  | TEXT    | Last departure timestamp         |

### `user_locations`
Real-time GPS locations from users.

| Column     | Type    | Description                    |
|------------|---------|--------------------------------|
| id         | INTEGER | Primary key                    |
| bus_id     | TEXT    | Associated bus                 |
| user_id    | TEXT    | Username                       |
| user_type  | TEXT    | 'driver' or 'student'          |
| lat        | REAL    | Latitude                       |
| lon        | REAL    | Longitude                      |
| accuracy   | REAL    | GPS accuracy in meters         |
| timestamp  | TEXT    | Update time (ISO 8601)         |
| cluster_id | INTEGER | Associated cluster ID          |
| weight     | REAL    | Weight for aggregation         |

*Note: (bus_id, user_id) is unique - replaces on conflict*

### `confirmations`
Student arrival confirmations.

| Column    | Type    | Description              |
|-----------|---------|--------------------------|
| id        | INTEGER | Primary key              |
| bus_id    | TEXT    | Bus identifier           |
| stop_id   | INTEGER | Stop ID                  |
| user_type | TEXT    | 'student' or 'driver'    |
| user_id   | TEXT    | Username                 |
| timestamp | TEXT    | Confirmation time        |

### `location_clusters`
Aggregated location clusters.

| Column       | Type    | Description                    |
|--------------|---------|--------------------------------|
| id           | INTEGER | Primary key                    |
| bus_id       | TEXT    | Bus identifier                 |
| lat          | REAL    | Cluster center latitude        |
| lon          | REAL    | Cluster center longitude       |
| radius       | REAL    | Cluster radius in meters       |
| point_count  | INTEGER | Points in this cluster         |
| total_points | INTEGER | Total points considered        |
| timestamp    | TEXT    | Creation time                  |
| is_majority  | BOOLEAN | Is this the majority cluster?  |

## 🚢 Deployment

### Production Checklist

- [ ] Change default user passwords
- [ ] Set strong `FLASK_SECRET_KEY` in environment variables
- [ ] Set `FLASK_ENV=production`
- [ ] Configure HTTPS/SSL certificates
- [ ] Set up proper database backups
- [ ] Configure firewall rules
- [ ] Enable rate limiting
- [ ] Set up logging and monitoring
- [ ] Test GPS permissions on target devices

### Heroku Deployment

1. Create `Procfile`:
```
web: gunicorn wsgi:application
```

2. Deploy:
```bash
heroku create your-app-name
git push heroku main
heroku config:set FLASK_SECRET_KEY=your-secret-key
```

### Docker Deployment

Create `Dockerfile`:
```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "wsgi:application"]
```

Build and run:
```bash
docker build -t bus-tracker .
docker run -p 8000:8000 -e FLASK_SECRET_KEY=your-key bus-tracker
```

## 🔧 Technical Details

### GPS-Based Location Tracking

The system uses driver GPS for real-time bus positioning:

1. **Driver GPS Priority**: Only driver's location updates bus position
2. **Proximity Detection**: Checks distance to all stops (40-meter radius)
3. **Automatic Stop Updates**: Changes to nearest stop when in proximity
4. **Continuous Tracking**: Updates every second while GPS is active

### Automatic Stop Detection

**Proximity-Based System:**
- **Detection Radius**: 40 meters from stop coordinates
- **Auto-arrival**: Status changes to "arrived" when within radius
- **Auto-departure**: Status changes to "departing" when outside all stop radii
- **Haversine Formula**: Accurate distance calculation using GPS coordinates

### Daily Reset System

**Automatic Midnight Reset:**
- Background thread calculates time until midnight
- Sleeps until 00:00 server time
- Resets bus to first stop (stop_index = 0)
- Clears all confirmations and status
- Repeats daily

### Fallback Manual Controls

**Smart Button Logic:**
- Manual buttons check GPS age before allowing action
- If GPS updated within 30 seconds: buttons show error
- If GPS older than 30 seconds: buttons work normally
- Prevents conflicting control methods

### Map Interaction

**User-Controlled Navigation:**
- No automatic map centering
- User can pan/zoom freely
- "Center on Bus" button provides optional recentering
- Smooth flyTo animation when centering
- Bus marker updates without affecting map view

### Session Management

- **Persistent Sessions**: 7-day lifetime
- **Multi-User Support**: UUID-based session IDs
- **Automatic Cleanup**: Removes inactive sessions
- **Secure Cookies**: HTTP-only, same-site protection

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Code Style

- Follow PEP 8 for Python code
- Use meaningful variable names
- Add comments for complex logic
- Write docstrings for functions

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors

- **Your Name** - Initial work

## 🙏 Acknowledgments

- Flask framework and community
- Leaflet.js for mapping functionality
- OpenStreetMap for map tiles
- Contributors and testers

## 📞 Support

For issues, questions, or contributions:
- Open an issue on GitHub
- Email: your.email@example.com

## 🗺️ Roadmap

### Version 3.1 - Current
- [x] Driver-only GPS location sharing
- [x] Automatic stop detection via proximity
- [x] Daily midnight reset system
- [x] Manual reset button for drivers
- [x] Fallback manual controls with GPS check
- [x] Map center control for students
- [x] Toast notifications for user feedback
- [x] "Coming Soon" message for student location sharing

### Planned Features
- [ ] Student location sharing (multi-user tracking)
- [ ] Push notifications for bus arrivals
- [ ] Historical route analytics
- [ ] Multiple bus support in UI
- [ ] Admin dashboard
- [ ] Mobile app (React Native)
- [ ] SMS/Email alerts
- [ ] Route optimization
- [ ] ETA calculations
- [ ] Offline mode support

---

**Note**: This system requires GPS-enabled devices and browser permission for geolocation. Currently, only drivers can share location. Students can view real-time bus position but cannot share their own location (feature coming soon). The bus automatically resets to the starting stop at midnight every day.
