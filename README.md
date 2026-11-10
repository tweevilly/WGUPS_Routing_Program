# WGUPS Routing Program
A Python delivery-routing simulator for the Western Governors University Parcel Service (WGUPS). It loads 40 packages onto three trucks, routes them with a greedy nearest-neighbor algorithm, and lets a supervisor check the status of any package at any time of day. It delivers every package within its constraints in 98.5 miles against a 140-mile limit.

Built as a **WGU Course C950: Data Structures and Algorithms II,**  project using plain Python 3.11 with no external libraries.

---

## The Problem

Packages on the Salt Lake City daily local delivery route were not arriving by their promised deadlines. The task: design an algorithm that delivers all 40 packages on time while honoring each package's special requirements and keeping combined truck mileage under 140 miles.

### Resources and rules

- 3 trucks, 2 drivers (only two trucks on the road at once); each truck carries up to 16 packages
- Trucks travel at an average of 18 mph, leave the hub no earlier than 8:00 AM, and may return to the hub to reload
- Loading and delivery are instantaneous; distances are symmetric

### Package Constraints

- Packages 6, 25, 28, 32 are delayed and arrive at the hub at 9:05 AM
- Packages 13, 14, 15, 16, 19, 20 must be delivered together
- Packages 3, 18, 36, 38 must go on Truck 2
- Package 15 has a 9:00 AM deadline; packages 1, 6, 13, 14, 16, 20, 25, 29, 30, 31, 34, 40 have a 10:30 AM deadline
- Package 9 has a wrong address until 10:20 AM, when it is corrected to 410 S. State St., Salt Lake City, UT 84111

---

## Data Sources and Extraction

### WGUPS Distance Table (.xlsx) :
- Converted to `CSV/Distance_File.csv`  
- Contains a symmetric matrix of distances between all delivery addresses  
- Used to calculate shortest paths between locations  
- Imported and parsed using the Python `csv` module

### WGUPS Package File (.xlsx) :
- Converted to `CSV/Package_File.csv`  
- Contains package IDs, addresses, deadlines, city/state/zip, weight, and special notes  
- Used to create package objects with proper constraints (e.g. delayed availability, truck assignment)  
- Loaded into a custom hash table for efficient lookup and status tracking

### Salt Lake City (.docx) :
- Downtown Map (used as visual reference, not directly parsed)

---

## Approach

**Custom Hash Table.** Package data lives in a hash table I implemented myself (linear probing) instead of a built-in dictionary. It supports constant-time lookup and status updates by package ID.

**Rule-Based Loading.** Packages are assigned to trucks in priority order:

1. Packages that must ride on Truck 2 are loaded first.
2. Delayed packages go on trucks that depart later.
3. Grouped packages are kept on the same truck.
4. Early-deadline packages (9:00 and 10:30 AM) are prioritized on Truck 1.
5. Remaining packages are balanced across trucks, up to the 16-package limit.

**Greedy nearest-neighbor delivery.** From its current location, each truck repeatedly drives to the closest delivery address. After every stop, mileage and time are updated at 18 mph and the delivery timestamp is saved on the package. Trucks 1 and 2 leave at 8:00 and 9:05 AM; Truck 3 leaves when a driver returns to the hub, so its start time is adjusted dynamically.

---
## Command-line interface

The menu lets a supervisor:

- View the status of one package or all packages at any time of day, including whether a package is delayed and when it was delivered
- See which packages are assigned to each truck
- See total mileage across all trucks and the time of the last delivery

---

## Project Structure 
The codebase is separated into distinct components, each with a single responsibility, making the system easier to understand, debug, and extend. 

```
Delivery_Project/    
├── Main.py                    # Entry point: routing logic, control flow, user interface
├── CreateHashMap.py           # Custom hash table for package lookup          
├── Package.py                 # Package data and time-based status logic
├── Truck.py                   # Truck capacity, speed, mileage, and current state
└── CSV/
    ├── Address_File.csv
    ├── Distance_File.csv
    └── Package_File.csv
```

The distance table and package file were provided by the course as spreadsheets and converted to CSV. They are read with Python's built-in csv module.

---

## How to run
Requires Python 3.11+. No packages to install.

```bash
git clone https://github.com/tweevilly/WGUPS_Routing_Program.git
cd WGUPS_Routing_Program/Delivery_Project
python Main.py
```

---

## Results

- All 40 packages delivered by their deadlines
- All "must be delivered together" and "Truck 2 only" requirements satisfied
- Package 9's address is corrected at 10:20 AM, and it is delivered at 11:36 AM
- Total mileage: 98.5 miles (limit: 140)
- Hash table lookups and status tracking verified

---

## Future improvements

- Load package constraints (truck assignments, delays, groupings) from the package data instead of hardcoding them, so the program can be reused for other cities
