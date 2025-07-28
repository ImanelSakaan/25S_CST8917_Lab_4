# 🚖 Real-Time Trip Monitoring Azure Function
### 📸 Demo Video 📹

🎥 Watch the demo here:  👉 🧪🔍🔁📁
**[▶️ YouTube Video Link](https://youtu.be/tT-EN_qwNzU)**

---
## 🔁 Booking Processing Workflow

Taxi dispatch networks generate large volumes of trip data in real time. To ensure service quality, safety, and operational insights, it's crucial to monitor this data as it arrives, analyze it immediately, and flag unusual patterns.

**We will implement a real-time event-driven system that:**
- Ingests taxi trip events from an Event Hub
- Uses an Azure Function to analyze trips for patterns (like group rides, cash payments, or suspiciously short rides)
- Routes this analysis through a Logic App
- Posts rich Adaptive Cards to Microsoft Teams to alert operations staff

**This allows dispatchers and supervisors to:**
- Immediately spot anomalies
- Monitor high-volume group rides
- Track vendors with suspicious activity
- Reduce manual review time

<div align="center">
  <img width="930" height="272" alt="image" src="https://github.com/user-attachments/assets/fe080c24-00db-49a1-a068-82f42c37f8d4" />
</div>


We create the following:
- **A namespace** is a container for all messaging components (queues and topics).
- **Azure Event Hubs** is a big data streaming platform and event ingestion service provided by Microsoft Azure.
- **Azure Logic Apps** enables you to automate workflows using a visual designer by connecting various services and triggers, both within Azure and from external systems like Microsoft Teams, Outlook, Salesforce, SQL, and Event Hubs.
- **Function App** to analyze incoming taxi trips for patterns (like group rides, cash payments, or suspiciously short rides), to detect unusual patterns in real-time.



## 📌 Tasks
### ✅ 1. Set Up Event Ingestion
  **1.1 Create Azure Resources:** (use JSON format)
- **Event Hub Namespace** is a container for all messaging components (queues and topics).
- **Azure Event Hubs** inside the namespace is a big data streaming platform and event ingestion service provided by Microsoft Azure.
- **Azure Logic Apps** (Consumption Plan) enables you to automate workflows using a visual designer by connecting various services and triggers, both within Azure and from external systems like Microsoft Teams, Outlook, Salesforce, SQL, and Event Hubs.

**1.2 Send Events:**
- Configure Azure Logic App to trigger When events are available in Event Hub (use batch mode).
This function receives JSON-formatted trip events from Event Hub (via Logic App), analyzes each trip based on distance, passenger count, and payment type, and returns structured insights.
<div align="center">  
  <img width="930" height="472" alt="pic06" src="https://github.com/user-attachments/assets/c1dfaf36-acfe-456f-8acd-d453a079c31d" />
</div>



### ✅ 2. Create Azure Function
**2.1 Create Azure Function App:**
- Use Python as the runtime stack.
- Deploy the following code to your Function:

```python
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
Absolutely! Here's a **flowchart** representing the logical flow of your **Azure Function trip analysis app**, from receiving input to producing the insights.

---

### 📊 Azure Function Trip Analysis – Flowchart

```mermaid
flowchart TD
    A[Start: HTTP Trigger from Logic App] --> B[Read JSON Payload]
    B --> C{Is Input a List?}
    C -- Yes --> D[Loop through each trip record]
    C -- No --> E[Wrap single trip in list] --> D

    D --> F[Extract trip details: vendorID, tripDistance, passengerCount, paymentType]

    F --> G[Analyze trip]
    G --> G1{tripDistance > 10?}
    G1 -- Yes --> H1[Add 'LongTrip' to insights]
    G1 -- No --> I1[Skip]

    G --> G2{passengerCount > 4?}
    G2 -- Yes --> H2[Add 'GroupRide']
    G2 -- No --> I2[Skip]

    G --> G3{paymentType == '2'?}
    G3 -- Yes --> H3[Add 'CashPayment']
    G3 -- No --> I3[Skip]

    G3 --> G4{paymentType == '2' AND tripDistance < 1?}
    G4 -- Yes --> H4[Add 'SuspiciousVendorActivity']
    G4 -- No --> I4[Skip]

    H1 & H2 & H3 & H4 --> J[Build response JSON with insights, summary, isInteresting]

    J --> K{More records?}
    K -- Yes --> D
    K -- No --> L[Return JSON array as HTTP response]

    L --> M[End]

```



### ✅ 3. Add Logic App Processing


<div align="center">  
  <img width="641" height="548" alt="pic52" src="https://github.com/user-attachments/assets/d0338834-14f1-4c6f-ac28-4ade671c935a" />
</div>




### ✅ 4. Post Adaptive Cards to Microsoft Teams

### 💬 yellowtaxiapp: is a Logic App that:
-  Listens to trip data (probably from Event Hubs or HTTP requests),
-  Sends it to an Azure Function for analysis,
-  Posts results to a Teams channel, email, or logs them.

  use the following post cards:
- Not Interesting Trip Card
- Interesting Trip Card
- Suspicious Vendor Activity

### ✅ 5. Microsoft Teams Test

<div align="center">  
  <img width="879" height="334" alt="image" src="https://github.com/user-attachments/assets/27bf6975-f5dc-4317-98be-b7e37f359667" />
</div>







