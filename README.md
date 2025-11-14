# Considition 2025 - EV Charging Bot

This repository contains a Python bot designed to compete in the Considition 2025 hackathon. [cite_start]The bot's goal is to provide charging recommendations to a fleet of electric vehicles (EVs) to maximize a final score[cite: 58].

[cite_start]The bot implements a **"Mandatory-First"** strategy, which is designed to fulfill a critical game rule: **a customer must charge at least once** to be eligible for any completion points[cite: 115, 174].

---

## 🧠 Algorithm Logic

The bot's logic is divided into two main phases: a one-time pre-computation and a tick-by-tick recommendation loop.

### 1. Initialization (Pre-computation)

Before the simulation starts, the bot performs two crucial setup steps:

1.  [cite_start]**Build Graph & Index Stations:** It parses the map data to build a graph representation of all nodes (locations) and edges (roads)[cite: 26]. [cite_start]During this process, it identifies and indexes all charging stations into a global `station_index` for quick lookup[cite: 42].
2.  [cite_start]**Compute All-Pairs Shortest Paths:** The bot uses the **Floyd-Warshall algorithm** to pre-compute the shortest path (and its distance) between *every single pair* of nodes on the map[cite: 68]. [cite_start]These results are stored in a global `path_cache`[cite: 70].

This heavy pre-computation allows the bot to make complex routing decisions in real-time (per-tick) without needing to run slow pathfinding algorithms repeatedly.

### 2. The "Mandatory-First" Strategy

The bot's core strategy is simple and robust: ensure every customer receives *exactly one* charging recommendation.

* [cite_start]A global set, `customer_has_been_routed_set`, tracks which customers have already been given a charging destination[cite: 17].
* On each tick, the bot iterates through all active customers.
* If a customer is already in the `customer_has_been_routed_set`, the bot **ignores them**.
* If a customer is new, the bot calls `find_best_mandatory_station` to find their optimal charging location.
* Once the best station is found, the bot issues a recommendation for that customer to charge there and immediately adds the customer's ID to the `customer_has_been_routed_set`.

[cite_start]This ensures every customer attempts to fulfill the "must charge once" rule [cite: 115] without being re-routed or given conflicting instructions later in their journey.

### 3. Finding the "Best" Station (The Brain)

[cite_start]The `find_best_mandatory_station` function [cite: 196] is the "brain" of the bot. It determines the single best station for a customer by iterating through **all** available stations on the map and calculating a `total_cost` for each.

[cite_start]The `total_cost` is a weighted sum of three factors, with weights determined by the customer's `persona`  [cite: 218, 19-24]:

1.  **Travel Cost:** The "detour" distance. This is the extra distance the customer must travel compared to driving straight to their destination (`(dist_to_station + dist_from_station) - direct_trip_distance`).
2.  **Time Cost:** A proxy for total time spent. This includes the estimated time to charge (to 80%) plus a penalty for stations that are busy or have broken chargers.
3.  **Energy Cost:** A score representing how "green" the station's energy is. [cite_start]The `calculate_green_score` function [cite: 161] [cite_start]analyzes the station's zone's energy sources (e.g., Solar, Wind, Coal) [cite: 177-191] [cite_start]and the time of day [cite: 171] to reward charging at more sustainable locations.

The station with the **lowest `total_cost`** is selected as the best mandatory station for that customer.

---

## 🚀 How to Use

Follow these steps to configure and run the bot for local testing or for an official cloud submission.

### 1. Prerequisites

* Python 3.x
* The `requests` library: `pip install requests`
* **Docker Desktop:** Required *only* for local testing to run the official game player[cite: 61].

### 2. Local Testing (Recommended)

Running locally allows you to test your algorithm without using API quota.

1.  **Pull & Run the Docker Player:**
    Open a terminal and run the following commands to get the official player[cite: 64, 68]:
    ```sh
    # Pull the latest player image
    docker pull considition/considition2025:latest

    # Run the player and map port 8080
    docker run -p 8080:8080 considition/considition2025
    ```

2.  **Configure `app.py`:**
    Open `app.py` and set the global constants for local testing:
    ```python
    # API Configuration
    API_KEY = ""  # Not needed for local
    BASE_URL = "http://localhost:8080" #

    # Map Configuration
    MAP_NAME = "Turbohill" # [cite: 52]
    MAP_SEED = "1760511906534" # [cite: 52]
    ```

### 3. Cloud Submission

When you are ready to submit to the official leaderboard:

1.  **Configure `app.py`:**
    Open `app.py` and set the constants for the cloud API:
    ```python
    # API Configuration
    API_KEY = "YOUR_PERSONAL_API_KEY_HERE" # [cite: 9, 19]
    [cite_start]BASE_URL = "[https://api.considition.com](https://api.considition.com)" # [cite: 10, 53]

    # Map Configuration
    MAP_NAME = "Turbohill" # Or the current official map
    MAP_SEED = "1760511906534" # Or the current official seed
    ```

### 4. Running the Bot

With your configuration set, simply run the `app.py` script:

```sh
python app.py
