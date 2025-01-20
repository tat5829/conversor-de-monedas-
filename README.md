import sqlite3
import datetime

# Conexión y configuración de la base de datos
def inicializar_bd():
    conexion = sqlite3.connect("literatura.db")
    cursor = conexion.cursor()

    # Crear tablas
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS libros (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            titulo TEXT UNIQUE,
            autor TEXT,
            idioma TEXT,
            anio_publicacion INTEGER
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS autores (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nombre TEXT UNIQUE,
            anio_nacimiento INTEGER,
            anio_fallecimiento INTEGER
        )
    ''')

    conexion.commit()
    return conexion

# Simulación de una API para buscar libros
def buscar_libro_api(titulo):
    # Base simulada de datos de libros
    api_libros = [
        {"titulo": "El Quijote", "autor": "Miguel de Cervantes", "idioma": "ES", "anio_publicacion": 1605},
        {"titulo": "Hamlet", "autor": "William Shakespeare", "idioma": "EN", "anio_publicacion": 1603},
        {"titulo": "Les Misérables", "autor": "Victor Hugo", "idioma": "FR", "anio_publicacion": 1862},
    ]

    for libro in api_libros:
        if libro["titulo"].lower() == titulo.lower():
            return libro
    return None

# Funciones de registro y consulta
def registrar_libro(conexion, libro):
    cursor = conexion.cursor()
    try:
        cursor.execute('''
            INSERT INTO libros (titulo, autor, idioma, anio_publicacion)
            VALUES (?, ?, ?, ?)
        ''', (libro["titulo"], libro["autor"], libro["idioma"], libro["anio_publicacion"]))

        cursor.execute('''
            INSERT OR IGNORE INTO autores (nombre, anio_nacimiento, anio_fallecimiento)
            VALUES (?, ?, ?)
        ''', (libro["autor"], None, None))

        conexion.commit()
        print(f"El libro '{libro['titulo']}' ha sido registrado exitosamente.")
    except sqlite3.IntegrityError:
        print(f"El libro '{libro['titulo']}' ya está registrado en la base de datos.")


def listar_libros(conexion):
    cursor = conexion.cursor()
    cursor.execute("SELECT titulo, autor, idioma, anio_publicacion FROM libros")
    libros = cursor.fetchall()

    if libros:
        print("\nLista de libros registrados:")
        for libro in libros:
            print(f"- {libro[0]} ({libro[3]}), Autor: {libro[1]}, Idioma: {libro[2]}")
    else:
        print("\nNo hay libros registrados en la base de datos.")


def listar_autores(conexion):
    cursor = conexion.cursor()
    cursor.execute("SELECT nombre FROM autores")
    autores = cursor.fetchall()

    if autores:
        print("\nLista de autores registrados:")
        for autor in autores:
            print(f"- {autor[0]}")
    else:
        print("\nNo hay autores registrados en la base de datos.")


def listar_autores_vivos(conexion, anio):
    cursor = conexion.cursor()
    cursor.execute('''
        SELECT nombre FROM autores
        WHERE (anio_nacimiento IS NOT NULL AND anio_nacimiento <= ?)
          AND (anio_fallecimiento IS NULL OR anio_fallecimiento > ?)
    ''', (anio, anio))

    autores = cursor.fetchall()

    if autores:
        print(f"\nAutores vivos en el año {anio}:")
        for autor in autores:
            print(f"- {autor[0]}")
    else:
        print(f"\nNo hay autores registrados que estuvieran vivos en el año {anio}.")


def listar_libros_por_idioma(conexion, idioma):
    cursor = conexion.cursor()
    cursor.execute("SELECT titulo, autor FROM libros WHERE idioma = ?", (idioma,))
    libros = cursor.fetchall()

    if libros:
        print(f"\nLibros en idioma {idioma}:")
        for libro in libros:
            print(f"- {libro[0]} (Autor: {libro[1]})")
    else:
        print(f"\nNo hay libros registrados en el idioma {idioma}.")

# Menú principal
def menu():
    conexion = inicializar_bd()

    while True:
        print("\n--- Menú Literalura ---")
        print("1. Buscar y registrar un libro por título")
        print("2. Listar todos los libros registrados")
        print("3. Listar todos los autores registrados")
        print("4. Listar autores vivos en un año específico")
        print("5. Listar libros por idioma")
        print("6. Salir")

        opcion = input("Seleccione una opción: ")

        if opcion == "1":
            titulo = input("Ingrese el título del libro a buscar: ")
            libro = buscar_libro_api(titulo)
            if libro:
                registrar_libro(conexion, libro)
            else:
                print(f"El libro '{titulo}' no se encontró en la API.")

        elif opcion == "2":
            listar_libros(conexion)

        elif opcion == "3":
            listar_autores(conexion)

        elif opcion == "4":
            anio = int(input("Ingrese el año: "))
            listar_autores_vivos(conexion, anio)

        elif opcion == "5":
            idioma = input("Ingrese el idioma (ES, EN, FR, PT): ").upper()
            listar_libros_por_idioma(conexion, idioma)

        elif opcion == "6":
            print("Gracias por usar Literalura. ¡Hasta luego!")
            break

        else:
            print("Opción no válida. Intente nuevamente.")

    conexion.close()

if __name__ == "__main__":
    menu()
