
import random
import tkinter as tk
from tkinter import messagebox


class Barco:
    def __init__(self, tamaño):
        self.tamaño = tamaño
        self.posiciones = []
        self.impactos = 0

    def colocar(self, fila, columna, horizontal=True):
        self.posiciones = []
        for i in range(self.tamaño):
            if horizontal:
                self.posiciones.append((fila, columna + i))
            else:
                self.posiciones.append((fila + i, columna))

    def recibir_impacto(self, fila, columna):
        if (fila, columna) in self.posiciones:
            self.impactos += 1
            return True
        return False

    def esta_hundido(self):
        return self.impactos == self.tamaño


class Tablero:
    def __init__(self, tamaño=10):
        self.tamaño = tamaño
        self.grid = [['~' for _ in range(tamaño)] for _ in range(tamaño)]
        self.barcos = []

    def es_valida(self, fila, columna):
        return 0 <= fila < self.tamaño and 0 <= columna < self.tamaño

    def agregar_barco(self, barco):
        for fila, columna in barco.posiciones:
            if not self.es_valida(fila, columna):
                return False
            if self.grid[fila][columna] != '~':
                return False

        for fila, columna in barco.posiciones:
            self.grid[fila][columna] = 'B'

        self.barcos.append(barco)
        return True

    def disparar(self, fila, columna):
        if not self.es_valida(fila, columna):
            return "Posición inválida"

        for barco in self.barcos:
            if barco.recibir_impacto(fila, columna):
                self.grid[fila][columna] = 'X'
                if barco.esta_hundido():
                    return "Barco hundido"
                return "Impacto"

        if self.grid[fila][columna] == '~':
            self.grid[fila][columna] = 'O'
            return "Agua"

        return "Ya disparaste aquí"

    def hay_barcos(self):
        for barco in self.barcos:
            if not barco.esta_hundido():
                return True
        return False


class Juego:
    def __init__(self):
        self.tablero = Tablero()

    def colocar_barcos_aleatorios(self):
        tamaños = [3, 2]
        for tamaño in tamaños:
            colocado = False
            while not colocado:
                barco = Barco(tamaño)
                fila = random.randint(0, 9)
                columna = random.randint(0, 9)
                horizontal = random.choice([True, False])
                barco.colocar(fila, columna, horizontal)
                colocado = self.tablero.agregar_barco(barco)


class InterfazBatallaNaval:
    def __init__(self, root):
        self.root = root
        self.root.title("Batalla Naval")

        self.juego = Juego()
        self.botones = []

        self.etiqueta_estado = tk.Label(
            root,
            text="Bienvenido a Batalla Naval",
            font=("Arial", 12)
        )
        self.etiqueta_estado.pack(pady=10)

        self.marco_tablero = tk.Frame(root)
        self.marco_tablero.pack()

        self.boton_nueva_partida = tk.Button(
            root,
            text="Nueva Partida",
            command=self.nueva_partida
        )
        self.boton_nueva_partida.pack(pady=10)

        self.crear_tablero()
        self.nueva_partida()

    def crear_tablero(self):
        for fila in range(10):
            fila_botones = []
            for columna in range(10):
                boton = tk.Button(
                    self.marco_tablero,
                    text=" ",
                    width=3,
                    height=1,
                    font=("Arial", 12),
                    command=lambda f=fila, c=columna: self.disparar(f, c)
                )
                boton.grid(row=fila, column=columna, padx=1, pady=1)
                fila_botones.append(boton)
            self.botones.append(fila_botones)

    def nueva_partida(self):
        self.juego = Juego()
        self.juego.colocar_barcos_aleatorios()
        self.etiqueta_estado.config(text="Nueva partida iniciada")

        for fila in range(10):
            for columna in range(10):
                boton = self.botones[fila][columna]
                boton.config(text=" ", state=tk.NORMAL)

    def disparar(self, fila, columna):
        resultado = self.juego.tablero.disparar(fila, columna)
        self.etiqueta_estado.config(text=resultado)

        boton = self.botones[fila][columna]

        if resultado in ["Impacto", "Barco hundido"]:
            boton.config(text="X", state=tk.DISABLED)
        elif resultado == "Agua":
            boton.config(text="O", state=tk.DISABLED)
        elif resultado == "Ya disparaste aquí":
            boton.config(state=tk.DISABLED)

        if not self.juego.tablero.hay_barcos():
            self.etiqueta_estado.config(text="Ganaste. Hundiste toda la flota.")
            messagebox.showinfo(
                "Victoria",
                "Ganaste. Hundiste toda la flota."
            )
            self.desactivar_tablero()

    def desactivar_tablero(self):
        for fila in range(10):
            for columna in range(10):
                self.botones[fila][columna].config(state=tk.DISABLED)


if __name__ == "__main__":
    root = tk.Tk()
    app = InterfazBatallaNaval(root)
    root.mainloop()

