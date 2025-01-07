# The Journey to Optimized Routing: A Step-by-Step Guide

Imagine you’re part of a logistics company in Morocco that needs to plan efficient delivery routes for customers while keeping costs low. Each day, vehicles navigate bustling cities like Casablanca and rural routes across diverse terrains. The challenge? You need a tool that calculates the best routes and assigns the right vehicles to the task. 

This guide will take you on a story-driven journey to deploy, test, and visualize an advanced route-planning and optimization system using OpenRouteService (ORS) and VROOM. By the end, you’ll have a working setup tailored to Morocco’s unique logistical needs.


## Setting the Scene

Every great journey starts with preparation. Before we can optimize routes, we need the right tools and resources.

### Gather Your Supplies
- **Docker and Docker Compose**: These are like the vehicles for our operation, helping us run ORS and VROOM in isolated, portable environments. Install them from the official [Docker website](https://docs.docker.com/get-docker/).
- **Morocco Map Data**: Our system needs a detailed map to calculate routes. Download the latest Morocco OSM file from [Geofabrik](https://download.geofabrik.de/).

### Organizing the Camp
Create a folder structure to keep your tools organized:
```plaintext
project/
├── docker-compose.yml    # Blueprint for the system
├── ors/
│   ├── files/
│   │   └── morocco-latest.osm.pbf  # The map data
├── vroom-conf/
│   └── config.yml        # VROOM's configuration
```
**Why?** Having a clear structure ensures the system knows where to find critical files, like maps and configurations, during setup.

## Building the Foundation

The core of our solution is the `docker-compose.yml` file, which defines how ORS and VROOM interact.

### The Blueprint
Our `docker-compose.yml` includes two services:
1. **OpenRouteService (ORS)**: Think of ORS as the navigator. It calculates routes using the Morocco map data.
2. **VROOM**: This is the strategist. It optimizes the routes provided by ORS, deciding which vehicle should take which job.

Here’s a snippet of the setup:
```yaml
services:
  ors:
    image: openrouteservice/openrouteservice:v8.0.0
    volumes:
      - ./ors/files/:/home/ors
    environment:
      ors.engine.source_file: /home/ors/morocco-latest.osm.pbf
      REBUILD_GRAPHS: "true"
  vroom:
    image: ghcr.io/vroom-project/vroom-docker:v1.14.0
    depends_on:
      - ors
```

**Why?** ORS needs the Morocco map file to create a "graph," a digital representation of roads, intersections, and routes. VROOM depends on ORS to calculate these routes.

## Starting the Adventure

With the foundation ready, it’s time to deploy the system.

### Bringing It to Life
Run this command in the terminal:
```bash
docker-compose up -d
```

This command launches the ORS and VROOM services. Check if they’re running:
```bash
docker ps
```

**What’s Happening?** ORS reads the Morocco map file to build routing graphs, while VROOM prepares to optimize jobs.

### **Input Description for VROOM**

To effectively utilize the VROOM API for vehicle routing optimization, it's essential to provide a well-structured JSON input that captures the problem's constraints, tasks, and resources. This section describes the input format and provides a detailed use case in Morocco, demonstrating how to model real-world delivery scenarios.

### **Input Structure**
The input JSON consists of three main components:

1. **Vehicles**:
   - Define the available fleet with properties like starting and ending locations, capacities, skills, and time windows.
   - Each vehicle is identified by a unique `id`.

2. **Jobs**:
   - Represent delivery or service tasks, each identified by a unique `id`.
   - Include properties like location, service time, priority, required skills, and time windows.

3. **Shipments** (Optional):
   - Define paired pickup and delivery tasks.
   - Useful when goods must be picked up at one location and delivered to another.


### **Example Use Case in Morocco**

#### **Scenario**:
- **Vehicles**:
  - Three delivery vans operating in Casablanca, Rabat, and Marrakesh.
  - Each van has specific skills (e.g., refrigeration or hazardous material handling).
- **Customers**:
  - Two customers with multiple delivery locations.
  - Each delivery has specific service times and skill requirements.


### **JSON Input**
Below is an example JSON input for the described scenario:

```json
{
  "vehicles": [
    {
      "id": 1,  // Unique ID for the first vehicle
      "profile": "driving-car",  // Routing profile
      "start": [-7.5898, 33.5731],  // Starting location: Casablanca
      "end": [-7.5898, 33.5731],  // Ending location: Casablanca (round trip)
      "capacity": [20],  // Capacity in volume or weight
      "skills": [1, 2],  // Skills: Refrigeration (1), General (2)
      "time_window": [28800, 64800]  // Available from 8:00 AM to 6:00 PM
    },
    {
      "id": 2,
      "profile": "driving-car",
      "start": [-6.8498, 34.0209],  // Starting location: Rabat
      "end": [-6.8498, 34.0209],  // Ending location: Rabat
      "capacity": [25],
      "skills": [1],  // Skills: Refrigeration only
      "time_window": [32400, 72000]  // Available from 9:00 AM to 8:00 PM
    },
    {
      "id": 3,
      "profile": "driving-car",
      "start": [-8.0089, 31.6345],  // Starting location: Marrakesh
      "end": [-8.0089, 31.6345],  // Ending location: Marrakesh
      "capacity": [30],
      "skills": [2],  // Skills: General only
      "time_window": [36000, 72000]  // Available from 10:00 AM to 8:00 PM
    }
  ],
  "jobs": [
    {
      "id": 1,  // Job ID
      "location": [-7.6126, 33.5671],  // Delivery location in Casablanca
      "service": 300,  // Service time: 5 minutes
      "delivery": [5],  // Delivery size: 5 units
      "skills": [1],  // Requires refrigeration
      "time_windows": [[32400, 43200]]  // Deliver between 9:00 AM and 12:00 PM
    },
    {
      "id": 2,
      "location": [-7.6269, 33.5693],  // Another location in Casablanca
      "service": 300,
      "delivery": [3],
      "skills": [2],  // General delivery
      "time_windows": [[36000, 46800]]  // Deliver between 10:00 AM and 1:00 PM
    },
    {
      "id": 3,
      "location": [-6.8348, 34.0331],  // Delivery in Rabat
      "service": 600,  // Service time: 10 minutes
      "delivery": [10],
      "skills": [1],
      "time_windows": [[39600, 54000]]  // Deliver between 11:00 AM and 3:00 PM
    },
    {
      "id": 4,
      "location": [-6.8534, 34.0132],  // Another delivery in Rabat
      "service": 300,
      "delivery": [7],
      "skills": [1],
      "time_windows": [[32400, 43200]]
    },
    {
      "id": 5,
      "location": [-8.0153, 31.6338],  // Delivery in Marrakesh
      "service": 300,
      "delivery": [4],
      "skills": [2],
      "time_windows": [[36000, 54000]]  // Deliver between 10:00 AM and 3:00 PM
    },
    {
      "id": 6,
      "location": [-8.0211, 31.6374],  // Another delivery in Marrakesh
      "service": 300,
      "delivery": [8],
      "skills": [2],
      "time_windows": [[39600, 57600]]  // Deliver between 11:00 AM and 4:00 PM
    }
  ]
}
```

### **Explanation of the Example**

1. **Vehicles**:
   - Vehicle 1 operates in Casablanca, supports refrigeration and general tasks, and can deliver up to 20 units per trip.
   - Vehicle 2 operates in Rabat, exclusively handles refrigeration, and can deliver up to 25 units.
   - Vehicle 3 operates in Marrakesh, handles general deliveries, and has a capacity of 30 units.

2. **Jobs**:
   - Deliveries in Casablanca, Rabat, and Marrakesh have unique time windows and service durations.
   - Each job specifies the required skills (e.g., refrigeration) and delivery size.

3. **Skills**:
   - Skills ensure compatibility between vehicles and tasks. For example:
     - Refrigeration deliveries (`skills: [1]`) require vehicles equipped with refrigeration capability.

