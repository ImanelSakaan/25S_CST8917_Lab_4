# 🚖 Real-Time Trip Monitoring Azure Function

This Azure Function analyzes incoming taxi trip events to detect unusual patterns in real-time. It is used as part of a larger Logic App workflow for a transportation dispatch system.
## 📹 Demo Video

👉 [Link to Demo Video](https://your-demo-video-link.com)

---
## 🔁 Example: Booking Processing Workflow

We create the following:
- **A namespace** is a container for all messaging components (queues and topics).
- **Azure Event Hubs** is a big data streaming platform and event ingestion service provided by Microsoft Azure.
- **Function App** to analyze trips for patterns (like group rides, cash payments, or suspiciously short rides).

<img width="930" height="272" alt="image" src="https://github.com/user-attachments/assets/fe080c24-00db-49a1-a068-82f42c37f8d4" />


## 📌 Function Purpose

This function receives JSON-formatted trip events from Event Hub (via Logic App), analyzes each trip based on distance, passenger count, and payment type, and returns structured insights.

## 🔍 Analysis Criteria

The function flags a trip as "interesting" if it meets any of the following:

- `LongTrip`: Distance > 10 miles
- `GroupRide`: More than 4 passengers
- `CashPayment`: Payment type is `2` (cash)
- `SuspiciousVendorActivity`: Cash payment and distance < 1 mile

## 🔁 Sample Input

```json
[
  {
    "ContentData": {
      "vendorID": "V001",
      "tripDistance": 0.6,
      "passengerCount": 2,
      "paymentType": "2"
    }
  }
]
````

## ✅ Sample Output

```json
[
  {
    "vendorID": "V001",
    "tripDistance": 0.6,
    "passengerCount": 2,
    "paymentType": "2",
    "insights": ["CashPayment", "SuspiciousVendorActivity"],
    "isInteresting": true,
    "summary": "2 flags: CashPayment, SuspiciousVendorActivity"
  }
]
```

## 🧪 How to Run Locally

1. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Run the function locally using Azure Functions Core Tools:

   ```bash
   func start
   ```

3. Send a test request:

   ```bash
   curl -X POST http://localhost:7071/api/analyze_trip \
        -H "Content-Type: application/json" \
        -d '[{"ContentData": {"vendorID": "V001", "tripDistance": 12, "passengerCount": 5, "paymentType": "2"}}]'
   ```

## 📁 Project Structure

```
/trip-analyzer-function/
│
├── __init__.py              # Main Azure Function code
├── function.json            # Trigger definition (optional if using VS Code)
├── requirements.txt         # Python dependencies
└── README.md                # This file
```

## 📬 Output Usage

The function output is passed to an Azure Logic App, which:

* Iterates over each result
* Posts an Adaptive Card to Microsoft Teams based on insights



```

Would you like a downloadable `.zip` or `.tar.gz` of these three files bundled for submission?
```











































##  Scenario: Real-Time Trip Monitoring for Taxi Dispatch System

Modern transportation systems—like ride-sharing platforms and taxi dispatch networks—generate large volumes of trip data in real time. To ensure service quality, safety, and operational insights, it's crucial to monitor this data as it arrives, analyze it immediately, and flag unusual patterns.

Imagine you're working for a transportation technology company that supports a large network of taxi services across the city. Your job is to help the operations team by automatically analyzing incoming trip data and notifying them about any trips that might be unusual or suspicious.

Your system will monitor data such as:

- Number of passengers
- Trip distance
- Payment method
- Vendor ID

You will implement a real-time event-driven system that:
- Ingests taxi trip events from an Event Hub
- Uses an Azure Function to analyze trips for patterns (like group rides, cash payments, or suspiciously short rides)
- Routes this analysis through a Logic App
- Posts rich Adaptive Cards to Microsoft Teams to alert operations staff

This allows dispatchers and supervisors to:
- Immediately spot anomalies
- Monitor high-volume group rides
- Track vendors with suspicious activity
- Reduce manual review time

Example Use Cases:
- A vendor frequently reports short trips (possible fraud)
- High trip volume with cash payment (potential tax evasion risk)
- Unusual passenger patterns (e.g., 5+ passengers on multiple rides)
---


## Tasks

### 1. Set Up Event Ingestion

- Create an Event Hub and simulate sending trip events (use JSON format).
- Configure Azure Logic App to trigger When events are available in Event Hub (use batch mode).

### 2. Create Azure Function

  ``` python
  import azure.functions as func
  import logging
  import json

  app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

  @app.route(route="")
  def analyze_trip(req: func.HttpRequest) -> func.HttpResponse:
      try:
          input_data = req.get_json()
          trips = input_data if isinstance(input_data, list) else [input_data]

          results = []

          for record in trips:
              trip = record.get("ContentData", {})  # ✅ Extract inner trip data

              vendor = trip.get("vendorID")
              distance = float(trip.get("tripDistance", 0))
              passenger_count = int(trip.get("passengerCount", 0))
              payment = str(trip.get("paymentType"))  # Cast to string to match logic

              insights = []

              if distance > 10:
                  insights.append("LongTrip")
              if passenger_count > 4:
                  insights.append("GroupRide")
              if payment == "2":
                  insights.append("CashPayment")
              if payment == "2" and distance < 1:
                  insights.append("SuspiciousVendorActivity")

              results.append({
                  "vendorID": vendor,
                  "tripDistance": distance,
                  "passengerCount": passenger_count,
                  "paymentType": payment,
                  "insights": insights,
                  "isInteresting": bool(insights),
                  "summary": f"{len(insights)} flags: {', '.join(insights)}" if insights else "Trip normal"
              })

          return func.HttpResponse(
              body=json.dumps(results),
              status_code=200,
              mimetype="application/json"
          )

      except Exception as e:
          logging.error(f"Error processing trip data: {e}")
          return func.HttpResponse(f"Error: {str(e)}", status_code=400)
  ```

### 3. Add Logic App Processing
- In Logic App, add a `For Each` loop over the function result.

- Add a Condition:
  - If `item()?['isInteresting']` is `true`, continue to Teams branching
  - Else, post a “Trip Analyzed – No Issues” card

### 4. Post Adaptive Cards to Microsoft Teams

Use the following post cards:
- Not Interesting Trip Card
  ```json
    {
    "type": "AdaptiveCard",
    "body": [
      {
        "type": "TextBlock",
        "text": "✅ Trip Analyzed - No Issues",
        "weight": "Bolder",
        "size": "Large",
        "color": "Good"
      },
      {
        "type": "FactSet",
        "facts": [
          { "title": "Vendor", "value": "@{items('For_each')?['vendorID']}" },
          { "title": "Distance (mi)", "value": "@{items('For_each')?['tripDistance']}" },
          { "title": "Passengers", "value": "@{items('For_each')?['passengerCount']}" },
          { "title": "Payment", "value": "@{items('For_each')?['paymentType']}" },
          { "title": "Summary", "value": "@{items('For_each')?['summary']}" }
        ]
      }
    ],
    "actions": [],
    "version": "1.2"
  }
  ```
- Interesting Trip Card
  ```json
    {
    "type": "AdaptiveCard",
    "body": [
      {
        "type": "TextBlock",
        "text": "🚨 Interesting Trip Detected",
        "weight": "Bolder",
        "size": "Large",
        "color": "Attention"
      },
      {
        "type": "FactSet",
        "facts": [
          { "title": "Vendor", "value": "@{items('For_each')?['vendorID']}" },
          { "title": "Distance (mi)", "value": "@{items('For_each')?['tripDistance']}" },
          { "title": "Passengers", "value": "@{items('For_each')?['passengerCount']}" },
          { "title": "Payment", "value": "@{items('For_each')?['paymentType']}" },
          { "title": "Insights", "value": "@{join(items('For_each')?['insights'], ', ')}" }
        ]
      }
    ],
    "actions": [],
    "version": "1.2"
  }
  ```

- Suspicious Vendor Activity
  ```json
    {
    "type": "AdaptiveCard",
    "body": [
      {
        "type": "TextBlock",
        "text": "⚠️ Suspicious Vendor Activity Detected",
        "weight": "Bolder",
        "size": "Large",
        "color": "Attention"
      },
      {
        "type": "FactSet",
        "facts": [
          { "title": "Vendor", "value": "@{items('For_each')?['vendorID']}" },
          { "title": "Distance (mi)", "value": "@{items('For_each')?['tripDistance']}" },
          { "title": "Passengers", "value": "@{items('For_each')?['passengerCount']}" },
          { "title": "Payment", "value": "@{items('For_each')?['paymentType']}" },
          { "title": "Insights", "value": "@{join(items('For_each')?['insights'], ', ')}" }
        ]
      }
    ],
    "actions": [],
    "version": "1.2"
  }
  ```


### Record a Demo

- Create a short video (3–5 minutes) demonstrating:
  - Your Logic App in action.
  - Explanation of your setup.
  - Any lessons learned.

---

## Deliverables

- Your completed Logic App workflow (.json export)
- A screenshot or exported image of your trip monitoring logic flowchart.
- Your written project report (can be a separate file or included in README.md) that explains:
  - Your architecture and Logic App steps
  - Description of your Azure Function logic
  - Example input/output
  - Any extra insights or improvements you suggest
- A demo video uploaded to YouTube, linked in the `README.md`.

---

## Submission Instructions

Submit the following via Brightspace:

- Link to your **public GitHub repository** containing:
  - Logic App definitions
  - Optional Function or Cognitive Service code
  - `README.md` with documentation
  - Your demo video link (if available)
- Your GitHub repo must be **well-structured and publicly accessible**.

**Deadline**: *Friday, 1 August 2025*


