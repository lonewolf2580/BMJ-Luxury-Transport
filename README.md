# 🚖 Luxury Ride-Sharing Platform (BMJ Transport)

## 📌 Overview  
BMJ is a **tech-enabled luxury transportation platform** designed to provide high-end ride services. This document details the **system architecture, tech stack, ride-matching algorithm, database schema, and workflow**.

---

## 🏗 System Architecture  

```plaintext
                    ┌──────────────────────────────┐
                    │          User App            │
                    │   (React Native / Flutter)   │
                    └──────────▲─────────────┬────┘
                               │             │
           ┌───────────────────┘             │
           │                           ┌─────▼───────┐
           │                           │   Driver App │
           │                           │(React Native)│
           │                           └─────▲───────┘
           │                                 │
 ┌─────────▼────────┐                   ┌───┴────┐
 │      Backend     │  WebSocket/API     │ Matching│
 │  (Node.js/Django)│◄──────────────────►│ Engine │
 │                  │                    └───▲────┘
 └───┬───────┬──────┘                        │
     │       │          ┌────────────────────┘
     │       │          │
     │       ▼          ▼
 ┌───▼───┐  ┌──────────┴──────────┐
 │Database│  │Real-time GPS System│
 │(NoSQL) │  │   (Google Maps)   │
 └────▲───┘  └─────────▲──────────┘
      │                │
      │          ┌─────┴──────────┐
      │          │ Payment Gateway │
      │          │ (Stripe/Paystack) │
      │          └─────────────────┘
      │
      ▼
┌─────────────┐
│ Analytics &  │
│ ML Models   │
└─────────────┘
```

## Tech Stack

### Frontend (Mobile App)

| Framework | Libraries & Tools |
|-----------|-------------------|
| React Native | react-native-maps, react-native-geolocation-service, socket.io-client, react-native-push-notification, react-native-paystack, Redux/Recoil |
| Flutter | google_maps_flutter, geolocator, flutter_socket_io, firebase_messaging, flutterwave, Provider/BLoC |

### Backend

| Feature | Technology |
|---------|------------|
| Server | Node.js (Express/NestJS) or Django (Python) |
| Database | NoSQL (MongoDB, Firebase Firestore, DynamoDB) or SQL (PostgreSQL, MySQL) |
| Ride Matching | Google S2 / Uber H3 for geospatial indexing |
| Real-time Tracking | WebSockets, Firebase Realtime Database |
| Payments | Stripe, Paystack, Flutterwave |
| Push Notifications | Firebase Cloud Messaging, Twilio |
| Authentication | Firebase Auth, OAuth, JWT |

## Ride Matching Algorithm

1. **Rider Requests Ride**
    - Rider location is geocoded.
    - Backend saves the request.
2. **Find Nearby Drivers**
    - Query drivers table where status = online.
    - Use geospatial indexing (Google S2 / Uber H3) to find drivers within a 5 km radius.
3. **Rank Available Drivers**
    - ETA (shortest time to pickup).
    - Driver rating (higher is better).
    - Acceptance rate (drivers who reject often are deprioritized).
4. **Driver Notification**
    - Closest driver has 15 seconds to accept.
    - If declined, the next best driver is notified.
5. **Trip Confirmation & Real-time Tracking**
    - WebSockets / Firebase Realtime Database updates locations every 3-5 seconds.

##  Database Schema

###  Users

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique user ID |
| name | String | Full name |
| email | String | Unique email |
| phone | String | Contact number |
| role | Enum(rider, driver, admin) | User type |
| wallet_balance | Number | In-app balance |
| rating | Number | User’s rating (1-5) |
| created_at | Timestamp | Account creation |

###  Trips

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique trip ID |
| rider_id | UUID | Rider who booked |
| driver_id | UUID | Assigned driver |
| pickup_location | Object | {lat, long} format |
| dropoff_location | Object | {lat, long} format |
| status | Enum(requested, ongoing, completed, canceled) | Trip status |
| fare | Number | Total ride cost |
| payment_status | Enum(pending, paid, failed) | Payment state |

###  Drivers

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique driver ID |
| vehicle_info | Object | {make, model, plate_number} |
| status | Enum(online, offline, busy) | Driver’s availability |
| rating | Number | Driver’s average rating |
| current_location | Object | {lat, long} |

###  Payments

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Unique payment ID |
| trip_id | UUID | Related trip |
| user_id | UUID | Who made the payment |
| amount | Number | Payment amount |
| method | Enum(card, wallet, cash) | Payment method |
| status | Enum(pending, success, failed) | Payment status |

## Serverless vs Dedicated Backend

| Feature | Serverless (AWS Lambda, Firebase Functions) | Dedicated Backend (Node.js/Django) |
|---------|---------------------------------------------|-----------------------------------|
| Cost | Cheaper (pay per request) | Fixed hosting cost |
| Scalability | Auto-scales | Needs manual scaling |
| Performance | Slight latency | Faster response times |
| Complexity | Easier for MVP | More control |

## 🏗 Development Roadmap

###  Phase 1: MVP Development
- User authentication & profile setup
- Driver onboarding
- Basic ride booking & trip status updates
- Payment integration (Stripe, Paystack)

###  Phase 2: Real-time Tracking & Optimization
- Implement geospatial ride-matching
- Enable WebSocket/Firebase live tracking
- Add surge pricing

###  Phase 3: Advanced Features
- Ride history & analytics
- Machine learning for route optimization
- AI-based customer support