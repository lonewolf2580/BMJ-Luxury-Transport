#  Luxury Ride-Sharing Platform (BMJ Transport)

##  Overview  
BMJ is a **tech-enabled luxury transportation platform** designed to provide high-end ride services. This document details the **system architecture, tech stack, database schema, and workflow**.

---

##  System Architecture  

```plaintext
                    ┌──────────────────────────────┐
                    │          User App            │
                    │   (React Native / Flutter)   │
                    └──────────▲─────────────┬─────┘
                               │             │
           ┌───────────────────┘             │
           │                           ┌─────▼───────┐
           │                           │  Company    │
           │                           │ Dashboard   │
           │                           └─────▲───────┘
           │                                 │
 ┌─────────▼────────┐                    ┌───┴────┐
 │      Backend     │  WebSocket/API     │ Book   |
 │  (Node.js/Django)│◄──────────────────►│ Ride   │
 │                  │                    └───▲────┘
 └───┬───────┬──────┘                        │
     │       │          ┌────────────────────┘
     │       │          │
     │       ▼          ▼
 ┌───▼────┐  ┌──────────┴──────────┐
 │Database│  │Real-time GPS System │
 │(NoSQL) │  │   (Google Maps)     │
 └────▲───┘  └─────────▲───────────┘
      │                │
      │          ┌─────┴───────────┐
      │          │ Payment Gateway │
      │          │(Stripe/Paystack)│
      │          └─────────────────┘
      │
      ▼
┌─────────────┐
│ Analytics & │
│ ML Models   │
└─────────────┘
```
## Pricing Algorithm

The pricing for rides is directly proportional to flight costs from various sources. The fare calculation is as follows:

1. **Fetch Flight Prices**
    - Obtain flight prices from up to 4 sources.
    - Calculate the average flight price.

    ```python
    percentage = 50
    flight_average = (p1 + p2 + p3 + p4) / 4
    ride_cost = (flight_average / 100) * percentage
    ```

2. **Advanced Booking**
    - The fare remains consistent when booked in advance.

3. **Dynamic Pricing (3-5 Days Before Trip)**
    - Fare increases as the trip date approaches, based on airline prices.

4. **Same-Day Booking**
    - If all airlines are sold out, a higher price is set due to increased demand and no supply.

## API Integration for Flight Prices

Currently, there are no APIs available for Nigerian flight prices. Therefore, we will either need to directly contact airlines for their pricing data or manually update the flight prices in our system.

These APIs will help fetch real-time flight prices to calculate the ride cost accurately.

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
| Real-time Tracking | WebSockets, Firebase Realtime Database |
| Payments | Stripe, Paystack, Flutterwave |
| Push Notifications | Firebase Cloud Messaging, Twilio |
| Authentication | Firebase Auth, OAuth, JWT |

## Ride Booking Process

1. **User Schedules Ride**
    - The user selects the desired ride from a list of scheduled rides.
    - The user inputs the number of seats required and confirms the booking.
2. **Estimate Fare**
    - The system calculates the fare based on the number of seats and the dynamic pricing factors.
3. **Confirm Booking**
    - Once the booking is confirmed, the user receives a confirmation with ride details.
4. **Payment Processing**
    - The user pays for the ride in advance.
    - Payment status is updated to reflect the successful transaction.
5. **Enable Real-time Tracking**
    - On the day of the ride, use WebSockets / Firebase Realtime Database to update the vehicle's location every 2-3 seconds.
6. **Complete Ride**
    - Upon reaching the destination, the trip status is updated to completed.
    - Users can provide feedback on the ride experience.

## Dynamic Pricing Factors

1. **Flight Cancellations**
    - Prices will be higher on days with a high number of flight cancellations.
2. **Advanced Booking**
    - The fare remains consistent when booked in advance.
3. **Same-Day Booking**
    - Higher prices are set for same-day bookings due to increased demand.

## Company Fleet

- All rides are conducted using company-owned luxury vehicles.
- Focus on providing a premium and consistent ride experience.


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
- Enable WebSocket/Firebase live tracking
- Add surge pricing

###  Phase 3: Advanced Features
- Ride history & analytics
- Machine learning for route optimization
- AI-based customer support