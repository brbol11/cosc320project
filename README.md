# cosc320project


from collections import deque


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
        self.graph = {}        # adjacency list
        self.stations = {}     # id >>> Station
        self.undo_stack = []   # stack for undo


    # Graph Ops #

    def add_station(self, station_id, name, capacity):
        if station_id in self.stations:
            print("Station already exists ")
            return

        station = Station(station_id, name, capacity)
        self.stations[station_id] = station
        self.graph[station_id] = []

        self.undo_stack.append(("remove_station ", station_id))
        print("Station added ")

    def remove_station(self, station_id):
        if station_id not in self.graph:
            print("Station not found ")
            return

        del self.graph[station_id]
        del self.stations[station_id]

        for neighbors in self.graph.values():
            if station_id in neighbors:
                neighbors.remove(station_id)

        print("Station removed ")

    def add_connection(self, id1, id2):
        if id1 not in self.graph or id2 not in self.graph:
            print("Invalid station IDs ")
            return

        self.graph[id1].append(id2)
        self.graph[id2].append(id1)

        self.undo_stack.append(("remove_connection ", id1, id2))
        print("Connection added ")

    def remove_connection(self, id1, id2):
        if id2 in self.graph[id1]:
            self.graph[id1].remove(id2)
        if id1 in self.graph[id2]:
            self.graph[id2].remove(id1)

        print("Connection removed ")

    def display_network(self):
        print("\nTransit Network: ")
        for station_id in self.graph:
            neighbors = [self.stations[n].name for n in self.graph[station_id]]
            print(f"{self.stations[station_id].name} -> {neighbors}")


    # DFS #

    def dfs(self, start, visited=None):
        if visited is None:
            visited = set()

        visited.add(start)
        print(self.stations[start].name)

        for neighbor in self.graph[start]:
            if neighbor not in visited:
                self.dfs(neighbor, visited)


    # BFS #

    def bfs(self, start):
        visited = set()
        queue = deque([start])

        while queue:
            node = queue.popleft()

            if node not in visited:
                print(self.stations[node].name)
                visited.add(node)

                for neighbor in self.graph[node]:
                    if neighbor not in visited:
                        queue.append(neighbor)


    # Shortest Path (BFS) #

    def shortest_path(self, start, end):
        queue = deque([(start, [start])])
        visited = set()

        while queue:
            current, path = queue.popleft()

            if current == end:
                return path

            visited.add(current)

            for neighbor in self.graph[current]:
                if neighbor not in visited:
                    queue.append((neighbor, path + [neighbor]))

        return None


    # Passenger Simulation #

    def add_passenger(self, station_id, name):
        if station_id not in self.stations:
            print("Station not found.")
            return

        self.stations[station_id].passengers.append(name)
        print("Passenger added.")

    def board_passenger(self, station_id):
        if self.stations[station_id].passengers:
            passenger = self.stations[station_id].passengers.pop(0)
            print(f"{passenger} boarded.")
        else:
            print("No passengers waiting.")


    # Sorting #

    def bubble_sort_stations(self):
        station_list = list(self.stations.values())

        for i in range(len(station_list)):
            for j in range(0, len(station_list) - i - 1):
                if station_list[j].capacity > station_list[j + 1].capacity:
                    station_list[j], station_list[j + 1] = station_list[j + 1], station_list[j]

        return station_list


    # Searching #

    def linear_search(self, station_id):
        for station in self.stations.values():
            if station.station_id == station_id:
                return station
        return None

    def binary_search(self, station_list, target_id):
        low = 0
        high = len(station_list) - 1

        while low <= high:
            mid = (low + high) // 2

            if station_list[mid].station_id == target_id:
                return station_list[mid]
            elif station_list[mid].station_id < target_id:
                low = mid + 1
            else:
                high = mid - 1

        return None


    # Undo Feature #

    def undo(self):
        if not self.undo_stack:
            print("Nothing to undo.")
            return

        action = self.undo_stack.pop()

        if action[0] == "remove_station":
            self.remove_station(action[1])
        elif action[0] == "remove_connection":
            self.remove_connection(action[1], action[2])



# Menu System #

def main():
    system = TransitSystem()

    while True:
        print("\n--- Transit System Menu ---")
        print("1. Add Station")
        print("2. Add Connection")
        print("3. Display Network")
        print("4. DFS")
        print("5. BFS")
        print("6. Shortest Path")
        print("7. Add Passenger")
        print("8. Board Passenger")
        print("9. Sort Stations by Capacity")
        print("10. Undo")
        print("11. Exit")

        choice = input("Enter choice: ")

        if choice == "1":
            sid = int(input("Station ID: "))
            name = input("Name: ")
            cap = int(input("Capacity: "))
            system.add_station(sid, name, cap)

        elif choice == "2":
            id1 = int(input("From ID: "))
            id2 = int(input("To ID: "))
            system.add_connection(id1, id2)

        elif choice == "3":
            system.display_network()

        elif choice == "4":
            start = int(input("Start ID: "))
            system.dfs(start)

        elif choice == "5":
            start = int(input("Start ID: "))
            system.bfs(start)

        elif choice == "6":
            start = int(input("Start ID: "))
            end = int(input("End ID: "))
            path = system.shortest_path(start, end)
            print("Path:", path)

        elif choice == "7":
            sid = int(input("Station ID: "))
            name = input("Passenger Name: ")
            system.add_passenger(sid, name)

        elif choice == "8":
            sid = int(input("Station ID: "))
            system.board_passenger(sid)

        elif choice == "9":
            sorted_list = system.bubble_sort_stations()
            for s in sorted_list:
                print(s)

        elif choice == "10":
            system.undo()

        elif choice == "11":
            print("Exit")
            break

        else:
            print("Invalid choice.")


if __name__ == "__main__":
    main()
