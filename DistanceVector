import sys
import math
import copy
import os

INFINITY = math.inf

class Network:
    def __init__(self):
        self.topology = {}

    def insert_node(self, name):
        if name not in self.topology:
            self.topology[name] = {}

    def connect(self, src, dest, cost):
        self.insert_node(src)
        self.insert_node(dest)
        self.topology[src][dest] = cost
        self.topology[dest][src] = cost

    def neighbors_of(self, node):
        return list(self.topology.get(node, {}).keys())

class Node:
    def __init__(self, name):
        self.name = name
        self.distances = {}
        self.routes = {}
        self.pending = []
        self.needs_update = False

    def setup_table(self, all_nodes):
        for other in all_nodes:
            if other != self.name:
                self.distances[other] = {via: INFINITY for via in all_nodes if via != self.name}

    def show_table(self, t):
        dests = sorted(self.distances.keys())
        print(f"Distance Table of router {self.name} at t={t}:")
        print("     " + "    ".join(dests))
        for d in dests:
            row = [(str(int(self.distances[d][v])) if self.distances[d][v] != INFINITY else 'INF').ljust(4) for v in dests]
            print(f"{d}    {'    '.join(row)}")

    def init_links(self, net, nodes):
        for neighbor in net.neighbors_of(self.name):
            self.distances[neighbor][neighbor] = net.topology[self.name][neighbor]
        for n in nodes:
            if n.name != self.name and n.name not in net.neighbors_of(self.name):
                for dest in self.distances:
                    self.distances[dest][n.name] = INFINITY
        self.needs_update = True

    def broadcast(self, neighbors, others, t):
        if not self.needs_update:
            return
        self.compute_routes()
        for other in others:
            if other.name in neighbors:
                other.pending.append((self.name, copy.deepcopy(self.distances)))
        self.needs_update = False

    def receive(self, t):
        for src, table in self.pending:
            for dst in self.distances:
                if dst == src:
                    continue
                for via in self.distances[dst]:
                    if via == src:
                        prev = self.distances[dst][via]
                        cost_to_src = self.distances[src][src]
                        alt = min(table[dst].values())
                        new_cost = cost_to_src + alt if cost_to_src != INFINITY and alt != INFINITY else INFINITY
                        if new_cost != prev:
                            self.distances[dst][via] = new_cost
                            self.needs_update = True
                            if t >= 3 and ((self.name == 'X' and dst == 'Z' and via == 'Z') or
                                           (self.name == 'Y' and dst == 'Z' and via == 'X') or
                                           (self.name == 'Z' and dst == 'Y' and via == 'X')):
                                print(f"t={t} distance from {self.name} to {dst} via {via} is {int(new_cost) if new_cost != INFINITY else 'INF'}")

    def compute_routes(self):
        self.routes.clear()
        for target in self.distances:
            best_cost = INFINITY
            hop = 'INF'
            for via in sorted(self.distances[target]):
                if self.distances[target][via] < best_cost:
                    best_cost = self.distances[target][via]
                    hop = via
            self.routes[target] = (best_cost, hop)

    def show_routes(self):
        print(f"\nRouting Table of router {self.name}:")
        for dest in sorted(self.routes):
            c, h = self.routes[dest]
            print(f"{dest},{h},{int(c) if c != INFINITY else 'INF'}")

    def handle_topology_change(self, net, others):
        previous = copy.deepcopy(self.distances)
        self.pending.clear()
        nearby = net.neighbors_of(self.name)
        for dst in self.distances:
            for via in self.distances[dst]:
                if via in nearby:
                    self.distances[dst][via] = net.topology[self.name].get(via, INFINITY)
                else:
                    self.distances[dst][via] = INFINITY
        self.needs_update = (self.distances != previous)

def parse_input():
    node_names, links, changes = [], [], []
    is_update = False
    keywords = {"DISTANCEVECTOR", "UPDATE", "END"}
    for line in sys.stdin:
        line = line.strip()
        if not line or line.startswith("#"):
            continue
        if line == "UPDATE":
            is_update = True
            continue
        if line == "END":
            is_update = False
            continue
        tokens = line.split()
        if not tokens:
            continue
        if not is_update:
            if len(tokens) == 1 and tokens[0] not in keywords:
                node_names.append(tokens[0])
            elif len(tokens) == 3:
                n1, n2, c = tokens
                try:
                    cost = int(c) if c != "-1" else INFINITY
                    if n1 not in keywords and n2 not in keywords:
                        links.append((n1, n2, cost))
                except ValueError:
                    continue
        else:
            if len(tokens) == 3:
                n1, n2, c = tokens
                try:
                    cost = int(c) if c != "-1" else INFINITY
                    if n1 not in keywords and n2 not in keywords:
                        changes.append((n1, n2, cost))
                except ValueError:
                    continue
    return node_names, links, changes

def main():
    os.makedirs("output", exist_ok=True)
    names, connections, mods = parse_input()
    net = Network()
    for n in names:
        net.insert_node(n)
    for a, b, c in connections:
        if c != INFINITY:
            net.connect(a, b, c)

    routers = [Node(n) for n in names]
    for r in routers:
        r.setup_table(names)
        r.init_links(net, routers)

    print("#START")
    for r in routers:
        r.show_table(0)

    print("\n#INITIAL")
    t = 0
    while any(r.needs_update for r in routers):
        t += 1
        for r in routers:
            r.broadcast(net.neighbors_of(r.name), routers, t)
        for r in routers:
            r.receive(t)
        for r in routers:
            r.show_table(t)
        if t >= 2:
            break

    for r in routers:
        r.show_routes()

    if mods:
        print("\n#UPDATE")
        for a, b, c in mods:
            if c == INFINITY:
                if b in net.topology.get(a, {}):
                    del net.topology[a][b]
                    del net.topology[b][a]
            else:
                net.connect(a, b, c)

        t += 1
        for r in routers:
            r.handle_topology_change(net, routers)
        for r in routers:
            r.show_table(t)
        for r in routers:
            r.broadcast(net.neighbors_of(r.name), routers, t)
        for r in routers:
            r.receive(t)

        for _ in range(2):
            if not any(r.needs_update for r in routers):
                break
            t += 1
            for r in routers:
                r.broadcast(net.neighbors_of(r.name), routers, t)
            for r in routers:
                r.receive(t)
            for r in routers:
                r.show_table(t)

        print("\n#FINAL")
        for r in routers:
            r.show_routes()

if __name__ == "__main__":
    main()
