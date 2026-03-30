# cosc320project

from collections import deque
import heapq


# Station #

class Station:
    def __init__(self, station_id, name, capacity):
        self.station_id = station_id
        self.name = name
        self.capacity = capacity
        self.passengers = []  # queue

    def __str__(self):
        return f"{self.name} (ID: {self.station_id}, Capacity: {self.capacity})"


# Transit System #

class TransitSystem:
    def __init__(self):
        self.graph = {}        # adjacency list: {id: [(neighbor, distance)]}
        self.stations = {}     # id -> Station
        self.undo_stack = []


    # -------------------------
    # Graph Operations
    # -------------------------

    def add_station(self, station_id, name, capacity):
        if station_id in self.stations:
            print("Station already exists")
            return

        station = Station(station_id, name, capacity)
        self.stations[station_id] = station
        self.graph[station_id] = []

        print("Station added")

    def add_connection(self, id1, id2, distance):
        if id1 not in self.graph or id2 not in self.graph:
            print("Invalid station IDs")
            return

        self.graph[id1].append((id2, distance))
        self.graph[id2].append((id1, distance))

        print("Connection added")

    def display_network(self):
        print("\nTransit Network:")
        for station_id in self.graph:
            neighbors = [
                f"{self.stations[n].name} ({d})"
                for n, d in self.graph[station_id]
            ]
            print(f"{self.stations[station_id].name} -> {neighbors}")


    # -------------------------
    # DFS
    # -------------------------

    def dfs(self, start, visited=None):
        if visited is None:
            visited = set()

        visited.add(start)
        print(self.stations[start].name)

        for neighbor, _ in self.graph[start]:
            if neighbor not in visited:
                self.dfs(neighbor, visited)


    # -------------------------
    # BFS
    # -------------------------

    def bfs(self, start):
        visited = set()
        queue = deque([start])

        while queue:
            node = queue.popleft()

            if node not in visited:
                print(self.stations[node].name)
                visited.add(node)

                for neighbor, _ in self.graph[node]:
                    if neighbor not in visited:
                        queue.append(neighbor)


    # -------------------------
    # Traffic Adjustment
    # -------------------------

    def get_traffic_weight(self, base_distance, station_id):
        traffic = len(self.stations[station_id].passengers)
        return base_distance + traffic


    # -------------------------
    # Dijkstra (Shortest Path)
    # -------------------------

    def dijkstra(self, start, end):
        pq = [(0, start, [])]  # (distance, node, path)
        visited = set()

        while pq:
            dist, current, path = heapq.heappop(pq)

            if current in visited:
                continue

            path = path + [current]

            if current == end:
                return dist, path

            visited.add(current)

            for neighbor, weight in self.graph[current]:
                if neighbor not in visited:
                    adjusted_weight = self.get_traffic_weight(weight, neighbor)
                    heapq.heappush(pq, (dist + adjusted_weight, neighbor, path))

        return None


    # -------------------------
    # Helper: Print Path
    # -------------------------

    def print_path(self, path):
        return " -> ".join(self.stations[p].name for p in path)


    # -------------------------
    # Search by Name
    # -------------------------

    def search_by_name(self, name):
        for station in self.stations.values():
            if station.name.lower() == name.lower():
                return station.station_id
        return None


    # -------------------------
    # Passenger Simulation
    # -------------------------

    def add_passenger(self, station_id, name):
        if station_id not in self.stations:
            print("Station not found")
            return

        self.stations[station_id].passengers.append(name)
        print("Passenger added")

    def board_passenger(self, station_id):
        if self.stations[station_id].passengers:
            passenger = self.stations[station_id].passengers.pop(0)
            print(f"{passenger} boarded")
        else:
            print("No passengers waiting")


# -------------------------
# Menu System
# -------------------------

def main():
    system = TransitSystem()

    # Example Baltimore-style stations
    system.add_station(1, "Inner Harbor", 100)
    system.add_station(2, "Fells Point", 80)
    system.add_station(3, "Johns Hopkins", 120)
    system.add_station(4, "Downtown", 200)

    system.add_connection(1, 2, 5)
    system.add_connection(2, 3, 7)
    system.add_connection(1, 4, 3)
    system.add_connection(4, 3, 4)

    while True:
        print("\n--- Transit System Menu ---")
        print("1. Display Network")
        print("2. DFS")
        print("3. BFS")
        print("4. Shortest Path (Dijkstra)")
        print("5. Add Passenger (Traffic)")
        print("6. Board Passenger")
        print("7. Search Station by Name")
        print("8. Exit")

        choice = input("Enter choice: ")

        if choice == "1":
            system.display_network()

        elif choice == "2":
            start = int(input("Start ID: "))
            system.dfs(start)

        elif choice == "3":
            start = int(input("Start ID: "))
            system.bfs(start)

        elif choice == "4":
            start = int(input("Start ID: "))
            end = int(input("End ID: "))
            result = system.dijkstra(start, end)

            if result:
                dist, path = result
                print("Shortest Path:", system.print_path(path))
                print("Total Distance (with traffic):", dist)
            else:
                print("No path found")

        elif choice == "5":
            sid = int(input("Station ID: "))
            name = input("Passenger Name: ")
            system.add_passenger(sid, name)

        elif choice == "6":
            sid = int(input("Station ID: "))
            system.board_passenger(sid)

        elif choice == "7":
            name = input("Enter station name: ")
            sid = system.search_by_name(name)
            if sid:
                print(f"Station ID: {sid}")
            else:
                print("Not found")

        elif choice == "8":
            print("Exit")
            break

        else:
            print("Invalid choice")


if __name__ == "__main__":
    main()
