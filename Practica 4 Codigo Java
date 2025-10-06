// Practical4.java
// Compilar: javac Practical4.java
// Ejecutar: java Practical4

import java.io.*;
import java.util.*;

public class Practical4 {
    public static void main(String[] args) {
        System.out.println("=== Práctica #04 - Encuentro de vuelos baratos (Simulación) ===\n");

        // --- Parte 1: Cargar grafo de vuelos desde flights.txt ---
        String flightsFile = "flights.txt";
        FlightGraph fg = new FlightGraph();
        try {
            fg.loadFromFile(flightsFile);
        } catch (IOException e) {
            System.err.println("No se pudo leer " + flightsFile + ": " + e.getMessage());
            System.out.println("Creando ejemplo interno de vuelos...");
            fg.createExampleData();
        }

        System.out.println("Reporte: nodos y aristas cargadas:");
        fg.printReport();

        // --- Buscar ruta más barata: ejemplo ---
        String origen = "GYE"; // Guayaquil (ejemplo)
        String destino = "UIO"; // Quito (ejemplo)

        long t0 = System.nanoTime();
        Dijkstra.Result res = Dijkstra.shortestPath(fg, origen, destino);
        long t1 = System.nanoTime();

        System.out.println("\nBúsqueda de itinerario más barato de " + origen + " a " + destino + ":");
        if (!res.found) {
            System.out.println("No existe ruta desde " + origen + " a " + destino);
        } else {
            System.out.println("Costo total: $" + res.distance);
            System.out.println("Itinerario: " + String.join(" -> ", res.path));
        }
        System.out.printf("Tiempo de ejecución (Dijkstra): %.3f ms\n", (t1 - t0) / 1e6);

        // --- Parte 2: Mostrar dos gráficas de grafos desde archivos de notas ---
        System.out.println("\n--- Gráfica de grafos (ejemplo 1) desde graph1.txt ---");
        try { GraphVisualizer.loadAndShowGraph("graph1.txt"); } catch (Exception e) {
            System.out.println("No se encontró graph1.txt, mostrando ejemplo generado internamente.");
            GraphVisualizer.showExampleGraph1();
        }

        System.out.println("\n--- Gráfica de grafos (ejemplo 2) desde graph2.txt ---");
        try { GraphVisualizer.loadAndShowGraph("graph2.txt"); } catch (Exception e) {
            System.out.println("No se encontró graph2.txt, mostrando ejemplo generado internamente.");
            GraphVisualizer.showExampleGraph2();
        }

        // --- Parte 3: Gráfica de árboles (dos ejemplos) ---
        System.out.println("\n--- Árbol binario: Ejemplo 1 (tree1.txt) ---");
        try { BinaryTree.loadAndDemo("tree1.txt"); } catch (Exception e) {
            System.out.println("No se encontró tree1.txt, mostrando ejemplo interno.");
            BinaryTree.exampleDemo1();
        }

        System.out.println("\n--- Árbol binario: Ejemplo 2 (tree2.txt) ---");
        try { BinaryTree.loadAndDemo("tree2.txt"); } catch (Exception e) {
            System.out.println("No se encontró tree2.txt, mostrando ejemplo interno.");
            BinaryTree.exampleDemo2();
        }

        System.out.println("\n=== FIN DEL PROGRAMA ===\n");
    }

    // -------------------- Flight Graph --------------------
    static class FlightGraph {
        // adjacency list: origin -> list of edges
        Map<String, List<Edge>> adj = new HashMap<>();

        static class Edge {
            String to;
            double price;
            String flightCode;
            Edge(String to, double price, String flightCode) { this.to = to; this.price = price; this.flightCode = flightCode; }
        }

        void addEdge(String from, String to, double price, String flightCode) {
            adj.computeIfAbsent(from, k -> new ArrayList<>()).add(new Edge(to, price, flightCode));
            // ensure nodes exist
            adj.putIfAbsent(to, new ArrayList<>());
        }

        void loadFromFile(String filename) throws IOException {
            BufferedReader br = new BufferedReader(new FileReader(filename));
            String line;
            int count = 0;
            while ((line = br.readLine()) != null) {
                line = line.trim();
                if (line.isEmpty() || line.startsWith("#")) continue;
                // formato esperado: flightCode,from,to,price
                String[] parts = line.split(",");
                if (parts.length < 4) continue;
                String flightCode = parts[0].trim();
                String from = parts[1].trim();
                String to = parts[2].trim();
                double price = Double.parseDouble(parts[3].trim());
                addEdge(from, to, price, flightCode);
                count++;
            }
            br.close();
            System.out.println("Cargadas " + count + " aristas desde " + filename);
        }

        void createExampleData() {
            // Datos de ejemplo (ficticios)
            addEdge("GYE","UIO", 80.0, "FL100");
            addEdge("GYE","CUE", 60.0, "FL101");
            addEdge("CUE","UIO", 40.0, "FL102");
            addEdge("GYE","UIO", 140.0, "FL103"); // directo más caro
            addEdge("UIO","GYE", 75.0, "FL104");
            addEdge("GYE","SNC", 120.0, "FL105");
            addEdge("SNC","UIO", 30.0, "FL106");
        }

        void printReport() {
            System.out.println("Nodos: " + adj.keySet());
            System.out.println("Aristas (origen -> destino [precio, vuelo]):");
            for (String from : adj.keySet()) {
                for (Edge e : adj.get(from)) {
                    System.out.println("  " + from + " -> " + e.to + " [$" + e.price + ", " + e.flightCode + "]");
                }
            }
        }
    }

    // -------------------- Dijkstra --------------------
    static class Dijkstra {
        static class Result {
            boolean found;
            double distance;
            List<String> path;
            Result(boolean f, double d, List<String> p) { found = f; distance = d; path = p; }
        }

        static Result shortestPath(FlightGraph graph, String source, String target) {
            Map<String, Double> dist = new HashMap<>();
            Map<String, String> prev = new HashMap<>();
            for (String node : graph.adj.keySet()) dist.put(node, Double.POSITIVE_INFINITY);
            if (!dist.containsKey(source) || !dist.containsKey(target)) return new Result(false, Double.POSITIVE_INFINITY, new ArrayList<>());
            dist.put(source, 0.0);
            PriorityQueue<String> pq = new PriorityQueue<>(Comparator.comparingDouble(dist::get));
            pq.add(source);

            while (!pq.isEmpty()) {
                String u = pq.poll();
                double du = dist.get(u);
                if (u.equals(target)) break;
                for (FlightGraph.Edge e : graph.adj.getOrDefault(u, Collections.emptyList())) {
                    double alt = du + e.price;
                    if (alt < dist.getOrDefault(e.to, Double.POSITIVE_INFINITY)) {
                        dist.put(e.to, alt);
                        prev.put(e.to, u);
                        pq.remove(e.to); // safe remove attempt
                        pq.add(e.to);
                    }
                }
            }

            if (!prev.containsKey(target) && !source.equals(target)) {
                // maybe direct or none
                if (dist.get(target).isInfinite()) return new Result(false, Double.POSITIVE_INFINITY, new ArrayList<>());
            }

            // reconstruir camino
            List<String> path = new LinkedList<>();
            String cur = target;
            path.add(0, cur);
            while (prev.containsKey(cur)) {
                cur = prev.get(cur);
                path.add(0, cur);
            }
            if (!path.get(0).equals(source)) return new Result(false, Double.POSITIVE_INFINITY, new ArrayList<>());
            return new Result(true, dist.get(target), path);
        }
    }

    // -------------------- Graph Visualizer (ASCII simple) --------------------
    static class GraphVisualizer {
        // Formato de archivo esperado: cada línea "A B" indica arista A->B (sin pesos)
        static void loadAndShowGraph(String filename) throws IOException {
            BufferedReader br = new BufferedReader(new FileReader(filename));
            Map<String, Set<String>> adj = new HashMap<>();
            String line;
            while ((line = br.readLine()) != null) {
                line = line.trim();
                if (line.isEmpty() || line.startsWith("#")) continue;
                String[] p = line.split("\\s+");
                if (p.length >= 2) {
                    adj.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
                    adj.putIfAbsent(p[1], new HashSet<>());
                }
            }
            br.close();
            showAdjacency(adj);
        }

        static void showAdjacency(Map<String, Set<String>> adj) {
            System.out.println("Adyacencia del grafo:");
            for (String node : adj.keySet()) {
                System.out.println("  " + node + " -> " + adj.get(node));
            }
            System.out.println("\nRepresentación simple (lista):");
            for (String node : adj.keySet()) {
                System.out.print(node + ": ");
                for (String to : adj.get(node)) System.out.print(to + " ");
                System.out.println();
            }
        }

        static void showExampleGraph1() {
            Map<String, Set<String>> adj = new LinkedHashMap<>();
            adj.put("A", new LinkedHashSet<>(Arrays.asList("B","C")));
            adj.put("B", new LinkedHashSet<>(Arrays.asList("D")));
            adj.put("C", new LinkedHashSet<>(Arrays.asList("D","E")));
            adj.put("D", new LinkedHashSet<>(Arrays.asList("E")));
            adj.put("E", new LinkedHashSet<>());
            showAdjacency(adj);
        }

        static void showExampleGraph2() {
            Map<String, Set<String>> adj = new LinkedHashMap<>();
            adj.put("S", new LinkedHashSet<>(Arrays.asList("A","B")));
            adj.put("A", new LinkedHashSet<>(Arrays.asList("C")));
            adj.put("B", new LinkedHashSet<>(Arrays.asList("C","D")));
            adj.put("C", new LinkedHashSet<>(Arrays.asList("T")));
            adj.put("D", new LinkedHashSet<>(Arrays.asList("T")));
            adj.put("T", new LinkedHashSet<>());
            showAdjacency(adj);
        }
    }

    // -------------------- Binary Tree Example --------------------
    static class BinaryTree {
        static class Node {
            String val;
            Node left, right;
            Node(String v){ val = v; left = right = null; }
        }

        Node root;

        BinaryTree() { root = null; }

        static BinaryTree fromListLevelOrder(String[] vals) {
            BinaryTree tree = new BinaryTree();
            if (vals.length == 0) return tree;
            Queue<Node> q = new LinkedList<>();
            tree.root = new Node(vals[0]);
            q.add(tree.root);
            int i = 1;
            while (!q.isEmpty() && i < vals.length) {
                Node cur = q.poll();
                if (i < vals.length && !vals[i].equals("null")) {
                    cur.left = new Node(vals[i]);
                    q.add(cur.left);
                }
                i++;
                if (i < vals.length && !vals[i].equals("null")) {
                    cur.right = new Node(vals[i]);
                    q.add(cur.right);
                }
                i++;
            }
            return tree;
        }

        static void preorder(Node n, List<String> out) {
            if (n==null) return;
            out.add(n.val);
            preorder(n.left,out);
            preorder(n.right,out);
        }
        static void inorder(Node n, List<String> out) {
            if (n==null) return;
            inorder(n.left,out);
            out.add(n.val);
            inorder(n.right,out);
        }
        static void postorder(Node n, List<String> out) {
            if (n==null) return;
            postorder(n.left,out);
            postorder(n.right,out);
            out.add(n.val);
        }

        static void loadAndDemo(String filename) throws IOException {
            BufferedReader br = new BufferedReader(new FileReader(filename));
            String line = br.readLine();
            br.close();
            if (line == null) {
                System.out.println("Archivo vacío.");
                return;
            }
            // formato: nodos separados por coma en orden level-order; usar "null" para vacíos
            String[] vals = Arrays.stream(line.split(",")).map(String::trim).toArray(String[]::new);
            BinaryTree t = fromListLevelOrder(vals);
            demoTraversals(t);
        }

        static void demoTraversals(BinaryTree t) {
            List<String> pre = new ArrayList<>(), in = new ArrayList<>(), post = new ArrayList<>();
            preorder(t.root, pre); inorder(t.root, in); postorder(t.root, post);
            System.out.println("Preorden: " + pre);
            System.out.println("Inorden: " + in);
            System.out.println("Postorden: " + post);
        }

        static void exampleDemo1() {
            String[] vals = {"F","B","G","A","D","null","I","null","null","C","E","null","null","null","null"};
            BinaryTree t = fromListLevelOrder(vals);
            demoTraversals(t);
        }

        static void exampleDemo2() {
            String[] vals = {"1","2","3","4","5","6","7"};
            BinaryTree t = fromListLevelOrder(vals);
            demoTraversals(t);
        }
    }
}
