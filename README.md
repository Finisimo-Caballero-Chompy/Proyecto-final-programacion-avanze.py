import random

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

    def mostrar(self):
        for fila in self.grid:
            print(" ".join(fila))
        print()

    def es_valida(self, fila, columna):
        return 0 <= fila < self.tamaño and 0 <= columna < self.tamaño

    def agregar_barco(self, barco):
        for (fila, columna) in barco.posiciones:
            if not self.es_valida(fila, columna):
                return False
            if self.grid[fila][columna] != '~':
                return False

        for (fila, columna) in barco.posiciones:
            self.grid[fila][columna] = 'B'

        self.barcos.append(barco)
        return True

    def disparar(self, fila, columna):
        if not self.es_valida(fila, columna):
            print("Posición inválida")
            return

        for barco in self.barcos:
            if barco.recibir_impacto(fila, columna):
                print("Impacto")
                self.grid[fila][columna] = 'X'
                if barco.esta_hundido():
                    print("Barco hundido")
                return

        if self.grid[fila][columna] == '~':
            print("Agua")
            self.grid[fila][columna] = 'O'
        else:
            print("Ya disparaste aquí")

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

    def iniciar(self):
        print("Bienvenido a Batalla Naval\n")
        self.colocar_barcos_aleatorios()

        while self.tablero.hay_barcos():
            self.tablero.mostrar()
            try:
                fila = int(input("Fila: "))
                columna = int(input("Columna: "))
                self.tablero.disparar(fila, columna)
            except:
                print("Entrada inválida")

        print("Ganaste. Hundiste toda la flota.")


#profit
if __name__ == "__main__":
    juego = Juego()
    juego.iniciar()
