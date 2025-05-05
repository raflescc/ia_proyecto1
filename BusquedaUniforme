import tkinter as tk
from tkinter import ttk
import heapq
import matplotlib.pyplot as plt
import networkx as nx
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from matplotlib.patches import Patch

class BusquedaUniformeGUI:
    def __init__(self, root, grafo):
        self.root = root
        self.grafo = grafo
        self.pasos = []
        self.paso_actual = -1
        self.rutas_visuales = []
        self.camino_final = []
        self.costo_final = 0
        self.rutas_disponibles = []
        self.rutas_descartadas = []

        self.G = nx.Graph()
        for ciudad, conexiones in grafo.items():
            for destino, peso in conexiones.items():
                self.G.add_edge(ciudad, destino, weight=peso)

        self.posiciones = {
            "Arad": (0, 2.5), "Zerind": (0.75, 3.75), "Oradea": (1.5, 5),
            "Sibiu": (3, 2.5), "Timisoara": (0, 1), "Lugoj": (1.5, 0.5),
            "Mehadia": (2, -1), "Dobreta": (2, -2.5), "Craiova": (4, -2.5),
            "Rimnicu Vilcea": (3.5, 1), "Pitesti": (5.5, -0.5),
            "Fagaras": (5, 2.5), "Bucharest": (7.5, -1.5),
            "Giurgiu": (6.5, -2.5), "Urziceni": (9.5, -1), "Hirsova": (11.5, -1),
            "Eforie": (12, -2.5), "Vaslui": (11, 2), "Iasi": (10, 3), "Neamt": (8.5, 4)
        }

        self.root.title("Proyecto 1 - Búsqueda Uniforme")
        self.root.geometry("1200x700")
        self.configurar_interfaz()

    def configurar_interfaz(self):
        izquierda = tk.Frame(self.root)
        izquierda.pack(side=tk.LEFT, fill=tk.Y, padx=10, pady=10)

        ciudades = list(self.grafo.keys())

        tk.Label(izquierda, text="Ciudad de inicio:").pack()
        self.combo_inicio = ttk.Combobox(izquierda, values=ciudades, state="readonly")
        self.combo_inicio.pack()

        tk.Label(izquierda, text="Ciudad destino:").pack()
        self.combo_objetivo = ttk.Combobox(izquierda, values=ciudades, state="readonly")
        self.combo_objetivo.pack()

        tk.Button(izquierda, text="Iniciar búsqueda", command=self.iniciar_busqueda).pack(pady=10)
        tk.Button(izquierda, text="Paso siguiente →", command=self.paso_siguiente).pack(pady=2)
        tk.Button(izquierda, text="← Paso anterior", command=self.paso_anterior).pack(pady=2)
        tk.Button(izquierda, text="Salir", command=self.root.quit).pack(pady=20)

        centro = tk.Frame(self.root)
        centro.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

        self.figura = plt.Figure(figsize=(6, 5))
        self.ax = self.figura.add_subplot(111)
        self.canvas = FigureCanvasTkAgg(self.figura, master=centro)
        self.canvas.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=False)

        abajo = tk.Frame(centro)
        abajo.pack(side=tk.BOTTOM, fill=tk.X)
        self.texto_salida = tk.Text(abajo, width=10, height=20, font=("Courier", 10))
        self.texto_salida.pack(fill=tk.BOTH, expand=True)

    def iniciar_busqueda(self):
        inicio, objetivo = self.combo_inicio.get(), self.combo_objetivo.get()
        if inicio not in self.grafo or objetivo not in self.grafo:
            self.mostrar_texto("⚠️ Ciudades inválidas.")
            return

        self.pasos = self.generar_pasos_busqueda_uniforme(inicio, objetivo)
        self.paso_actual = -1
        self.paso_siguiente()

    def paso_siguiente(self):
        if self.paso_actual < len(self.pasos) - 1:
            self.paso_actual += 1
            self.mostrar_texto("\n".join(self.pasos[:self.paso_actual + 1]))
            self.actualizar_mapa()

    def paso_anterior(self):
        if self.paso_actual > 0:
            self.paso_actual -= 1
            self.mostrar_texto("\n".join(self.pasos[:self.paso_actual + 1]))
            self.actualizar_mapa()

    def mostrar_texto(self, texto):
        self.texto_salida.delete("1.0", tk.END)
        self.texto_salida.insert(tk.END, texto)
        self.texto_salida.see(tk.END)

    def actualizar_mapa(self):
        self.ax.clear()
        labels = nx.get_edge_attributes(self.G, 'weight')
        nx.draw(self.G, self.posiciones, ax=self.ax, with_labels=True, node_color='lightblue', node_size=700, font_size=9)

        # rutas disponibles (azul)
        for i in range(self.paso_actual + 1):
            if i < len(self.rutas_disponibles):
                rutas = self.rutas_disponibles[i]
                nx.draw_networkx_edges(self.G, self.posiciones, edgelist=rutas, ax=self.ax, edge_color='blue', width=2)

        # rutas descartadas (naranja)
        for i in range(self.paso_actual + 1):
            if i < len(self.rutas_descartadas):
                rutas = self.rutas_descartadas[i]
                nx.draw_networkx_edges(self.G, self.posiciones, edgelist=rutas, ax=self.ax, edge_color='orange', style='dashed', width=2)

        # ruta seleccionada (rojo)
        if self.paso_actual < len(self.rutas_visuales):
            ruta_actual = self.rutas_visuales[self.paso_actual]
            nx.draw_networkx_edges(self.G, self.posiciones, edgelist=ruta_actual, ax=self.ax, edge_color='red', width=2)

        # trayectoria final (verde)
        if self.paso_actual == len(self.pasos) - 1 and self.camino_final:
            final_edges = [(self.camino_final[i], self.camino_final[i+1]) for i in range(len(self.camino_final)-1)]
            nx.draw_networkx_edges(self.G, self.posiciones, edgelist=final_edges, ax=self.ax, edge_color='green', width=4)

        nx.draw_networkx_edge_labels(self.G, self.posiciones, edge_labels=labels, ax=self.ax, font_size=7, label_pos=0.5, rotate=False)

        leyenda = [
            Patch(color='blue', label='Rutas disponibles'),
            Patch(color='orange', label='Rutas descartadas'),
            Patch(color='red', label='Ruta seleccionada'),
            Patch(color='green', label='Trayectoria final'),
        ]
        self.ax.legend(handles=leyenda, loc='upper right', fontsize=9)
        self.figura.tight_layout()
        self.canvas.draw()

    def generar_pasos_busqueda_uniforme(self, inicio, objetivo):
        cola = [(0, [inicio])]
        visitadas = []
        mejor_costo = {inicio: 0}
        rutas_no_factibles_anterior = []
        rutas_no_factibles_actual = []
        paso = 0
        pasos_guardados = []
        self.rutas_visuales.clear()
        self.rutas_disponibles.clear()
        self.rutas_descartadas.clear()

        while cola:
            heapq.heapify(cola)
            costo_seleccionado, camino_seleccionado = heapq.nsmallest(1, cola)[0]
            ciudad_expandida = camino_seleccionado[-1]

            rutas_validas = []
            rutas_azules = []
            rutas_naranjas = []
            cola_filtrada = []

            for costo, camino in sorted(cola):
                destino = camino[-1]
                id_ruta = (tuple(camino), costo)

                if destino in visitadas:
                    continue
                elif destino in mejor_costo and costo > mejor_costo[destino]:
                    if id_ruta not in rutas_no_factibles_actual:
                        rutas_no_factibles_anterior.append((costo, camino))
                        rutas_no_factibles_actual.append(id_ruta)
                        rutas_naranjas += [(camino[i], camino[i+1]) for i in range(len(camino)-1)]
                    continue
                else:
                    rutas_validas.append((costo, camino))
                    cola_filtrada.append((costo, camino))

            cola = cola_filtrada.copy()

            salida = f"\nPaso {paso}\n\n"
            for costo, camino in rutas_validas:
                ciudades = ', '.join(camino)
                if (costo, camino) == (costo_seleccionado, camino_seleccionado):
                    salida += f"  →[{costo}] km: {ciudades}\n"
                else:
                    salida += f"    {costo}  km: {ciudades}\n"

                rutas_azules += [(camino[i], camino[i+1]) for i in range(len(camino)-1)]

            for costo, camino in rutas_no_factibles_anterior:
                ciudades = ', '.join(camino)
                salida += f"    ~{costo} km: {ciudades}---x\n"

            rutas_no_factibles_anterior = []

            salida += f"\nExpandidas: {', '.join(visitadas) if paso > 0 else 'ninguna'}\n"

            if ciudad_expandida == objetivo:
                salida += "\nTERMINA.\n\n"
                salida += (f"Trayectoria: {' → '.join(camino_seleccionado)}\n")
                salida += (f"Costo: {costo_seleccionado} km\n")
                pasos_guardados.append(salida)

                self.rutas_visuales.append(
                    [(camino_seleccionado[i], camino_seleccionado[i+1]) for i in range(len(camino_seleccionado)-1)]
                )
                self.rutas_disponibles.append(rutas_azules)
                self.rutas_descartadas.append(rutas_naranjas)
                self.camino_final = camino_seleccionado
                self.costo_final = costo_seleccionado
                break

            salida += "\n------------------------------------------------------------------------"
            pasos_guardados.append(salida)

            self.rutas_visuales.append(
                [(camino_seleccionado[i], camino_seleccionado[i+1]) for i in range(len(camino_seleccionado)-1)]
            )
            self.rutas_disponibles.append(rutas_azules)
            self.rutas_descartadas.append(rutas_naranjas)

            cola = [item for item in cola if item != (costo_seleccionado, camino_seleccionado)]

            if ciudad_expandida not in visitadas:
                visitadas.append(ciudad_expandida)
                for vecino, distancia in self.grafo[ciudad_expandida].items():
                    vecinos = self.grafo[ciudad_expandida].items()
                    if vecino in camino_seleccionado:
                        if len(vecinos) == 1:
                            rutas_no_factibles_anterior.append((costo_seleccionado, camino_seleccionado))
                            rutas_naranjas += [(camino_seleccionado[i], camino_seleccionado[i+1]) for i in range(len(camino_seleccionado)-1)]
                            self.rutas_descartadas.append(rutas_naranjas)
                        else:
                            ciudades = camino_seleccionado + [vecino]
                            id_ruta = (tuple(ciudades), costo_seleccionado + distancia)
                            if id_ruta not in rutas_no_factibles_actual:
                                rutas_no_factibles_actual.append(id_ruta)
                        continue

                    nuevo_costo = costo_seleccionado + distancia
                    nuevo_camino = camino_seleccionado + [vecino]
                    id_ruta = (tuple(nuevo_camino), nuevo_costo)

                    if vecino in mejor_costo and nuevo_costo >= mejor_costo[vecino]:
                        if id_ruta not in rutas_no_factibles_actual:
                            rutas_no_factibles_anterior.append((nuevo_costo, nuevo_camino))
                            rutas_naranjas += [(nuevo_camino[i], nuevo_camino[i+1]) for i in range(len(nuevo_camino)-1)]
                            rutas_no_factibles_actual.append(id_ruta)
                    else:
                        mejor_costo[vecino] = nuevo_costo
                        heapq.heappush(cola, (nuevo_costo, nuevo_camino))

            paso += 1

        return pasos_guardados

# Grafo
grafo = {
    "Arad": {"Sibiu": 140, "Timisoara": 118, "Zerind": 75},
    "Bucharest": {"Pitesti": 101, "Fagaras": 211, "Giurgiu": 90, "Urziceni": 85},
    "Craiova": {"Dobreta": 120, "Pitesti": 138, "Rimnicu Vilcea": 146},
    "Dobreta": {"Craiova": 120, "Mehadia": 75},
    "Eforie": {"Hirsova": 86},
    "Fagaras": {"Bucharest": 211, "Sibiu": 99},
    "Giurgiu": {"Bucharest": 90},
    "Hirsova": {"Eforie": 86, "Urziceni": 98},
    "Iasi": {"Neamt": 87, "Vaslui": 92},
    "Lugoj": {"Mehadia": 70, "Timisoara": 111},
    "Mehadia": {"Dobreta": 75, "Lugoj": 70},
    "Neamt": {"Iasi": 87},
    "Oradea": {"Sibiu": 151, "Zerind": 71},
    "Pitesti": {"Bucharest": 101, "Craiova": 138, "Rimnicu Vilcea": 97},
    "Rimnicu Vilcea": {"Craiova": 146, "Pitesti": 97, "Sibiu": 80},
    "Sibiu": {"Arad": 140, "Fagaras": 99, "Oradea": 151, "Rimnicu Vilcea": 80},
    "Timisoara": {"Arad": 118, "Lugoj": 111},
    "Urziceni": {"Bucharest": 85, "Hirsova": 98, "Vaslui": 142},
    "Vaslui": {"Iasi": 92, "Urziceni": 142},
    "Zerind": {"Arad": 75, "Oradea": 71}
}

if __name__ == "__main__":
    root = tk.Tk()
    app = BusquedaUniformeGUI(root, grafo)
    root.mainloop()
