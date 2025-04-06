# Calculadora-MN
Metodos Numericos, con interfaz tkinter, incluye Biseccion, Falsa Posicion, Newton-Raphson y Secante 
import tkinter as tk
from tkinter import messagebox, Menu
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import sympy as sp
import threading
from tkinter import ttk

x = sp.symbols('x')

def calcular_derivada(funcion):
    try:
        expr = sp.sympify(funcion)
        derivada = sp.diff(expr, x)
        return derivada
    except Exception as e:
        messagebox.showerror("Error", f"Error al calcular la derivada: {e}")
        return None

def evaluar_funcion(funcion, valor):
    try:
        expr = sp.sympify(funcion)
        return expr.evalf(subs={x: valor})
    except Exception as e:
        messagebox.showerror("Error", f"Error al evaluar la función: {e}")
        return None

def bisection(a, b, tol, funcion):
    if evaluar_funcion(funcion, a) * evaluar_funcion(funcion, b) >= 0:
        messagebox.showerror("Error", "La función debe tener signos opuestos en a y b.")
        return None
    iterations = []
    while (b - a) / 2.0 > tol:
        c = (a + b) / 2.0
        iterations.append((c, evaluar_funcion(funcion, c)))
        if evaluar_funcion(funcion, c) == 0:
            break
        elif evaluar_funcion(funcion, c) * evaluar_funcion(funcion, a) < 0:
            b = c
        else:
            a = c
    return iterations

def falsa_posicion(a, b, tol, funcion):
    if evaluar_funcion(funcion, a) * evaluar_funcion(funcion, b) >= 0:
        messagebox.showerror("Error", "La función debe tener signos opuestos en a y b.")
        return None
    iterations = []
    while abs(b - a) > tol:
        c = b - (evaluar_funcion(funcion, b) * (b - a)) / (evaluar_funcion(funcion, b) - evaluar_funcion(funcion, a))
        iterations.append((c, evaluar_funcion(funcion, c)))
        if evaluar_funcion(funcion, c) == 0:
            break
        elif evaluar_funcion(funcion, c) * evaluar_funcion(funcion, a) < 0:
            b = c
        else:
            a = c
    return iterations

def newton_raphson(x0, tol, funcion):
    iterations = []
    while True:
        fx = evaluar_funcion(funcion, x0)
        fpx = calcular_derivada(funcion).evalf(subs={x: x0})
        if fpx == 0:
            messagebox.showerror("Error", "La derivada es cero. No se puede continuar.")
            return None
        x1 = x0 - fx / fpx
        iterations.append((x1, evaluar_funcion(funcion, x1)))
        if abs(x1 - x0) < tol:
            break
        x0 = x1
    return iterations

def secante(x0, x1, tol, funcion):
    iterations = []
    while abs(x1 - x0) > tol:
        fx0 = evaluar_funcion(funcion, x0)
        fx1 = evaluar_funcion(funcion, x1)
        if fx1 - fx0 == 0:
            messagebox.showerror("Error", "División por cero en el método de la Secante.")
            return None
        x2 = x1 - fx1 * (x1 - x0) / (fx1 - fx0)
        iterations.append((x2, evaluar_funcion(funcion, x2)))
        x0, x1 = x1, x2
    return iterations

def habilitar_entradas(*args):
    metodo = metodo_var.get()
    entry_a.config(state="normal")
    entry_b.config(state="normal")
    entry_tol.config(state="normal")
    entry_x0.config(state="normal")
    entry_x1.config(state="normal")

    if metodo in ["Bisección", "Falsa Posición"]:
        entry_x0.config(state="disabled")
        entry_x1.config(state="disabled")
    elif metodo == "Newton-Raphson":
        entry_a.config(state="disabled")
        entry_b.config(state="disabled")
        entry_x1.config(state="disabled")
    elif metodo == "Secante":
        entry_a.config(state="disabled")
        entry_b.config(state="disabled")

def graficar_funcion():
    try:
        a = float(entry_a.get()) if entry_a.get() else -10
        b = float(entry_b.get()) if entry_b.get() else 10
        x_vals = np.linspace(a, b, 500)
        y_vals = [evaluar_funcion(funcion_usuario.get(), val) for val in x_vals]

        fig, ax = plt.subplots(figsize=(6, 4))
        ax.plot(x_vals, y_vals, label="f(x)", color="purple")  # Cambiado a lila
        ax.axhline(0, color="black", linewidth=0.8)
        ax.axvline(0, color="black", linewidth=0.8)
        ax.set_title("Gráfica de la función")
        ax.set_xlabel("x")
        ax.set_ylabel("f(x)")
        ax.legend()
        ax.grid()

        grafica_frame = tk.Toplevel(root)
        grafica_frame.title("Gráfica de la función")

        # Crear el widget de la gráfica con interacción habilitada
        canvas = FigureCanvasTkAgg(fig, master=grafica_frame)
        canvas.draw()
        canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)

        # Habilitar herramientas de navegación (zoom y movimiento)
        toolbar_frame = tk.Frame(grafica_frame)
        toolbar_frame.pack(fill=tk.X)
        toolbar = plt.matplotlib.backends.backend_tkagg.NavigationToolbar2Tk(canvas, toolbar_frame)
        toolbar.update()

    except Exception as e:
        messagebox.showerror("Error", f"Error al graficar: {e}")

def ejecutar_metodo():
    def calcular():
        boton_ejecutar.config(state="disabled")  # Deshabilitar el botón
        metodo = metodo_var.get()
        try:
            a = float(entry_a.get()) if entry_a.get() else None
            b = float(entry_b.get()) if entry_b.get() else None
            tol = float(entry_tol.get())
            x0 = float(entry_x0.get()) if entry_x0.get() else None
            x1 = float(entry_x1.get()) if entry_x1.get() else None

            if metodo == "Bisección":
                if a is None or b is None or a >= b:
                    messagebox.showerror("Error", "El valor de 'a' debe ser menor que 'b'.")
                    return
                resultado = bisection(a, b, tol, funcion_usuario.get())
            elif metodo == "Falsa Posición":
                if a is None or b is None or a >= b:
                    messagebox.showerror("Error", "El valor de 'a' y 'b' tienen que ser opuestos.")
                    return
                resultado = falsa_posicion(a, b, tol, funcion_usuario.get())
            elif metodo == "Newton-Raphson":
                if x0 is None:
                    messagebox.showerror("Error", "Debe ingresar un valor inicial x0.")
                    return
                resultado = newton_raphson(x0, tol, funcion_usuario.get())
            elif metodo == "Secante":
                if x0 is None or x1 is None:
                    messagebox.showerror("Error", "Debe ingresar dos valores iniciales x0 y x1.")
                    return
                resultado = secante(x0, x1, tol, funcion_usuario.get())
            else:
                messagebox.showerror("Error", "Método no reconocido.")
                return

            if resultado is not None:
                mostrar_resultados(resultado)
        except ValueError:
            messagebox.showerror("Error", "Por favor, ingrese valores numéricos válidos.")
        finally:
            boton_ejecutar.config(state="normal")  # Habilitar el botón nuevamente

    # Usar after para ejecutar los cálculos sin bloquear la interfaz
    root.after(100, calcular)

def mostrar_resultados(iterations):
    # Limpiar el marco de resultados
    for widget in frame_resultados.winfo_children():
        widget.destroy()

    if not iterations:
        tk.Label(frame_resultados, text="No se encontraron resultados.", font=("Arial", 10, "bold")).pack()
        return

    # Crear un Treeview para mostrar los resultados en forma de tabla
    tree = ttk.Treeview(frame_resultados, columns=("Iteración", "Raíz Aproximada", "f(Raíz)"), show="headings", height=10)
    tree.heading("Iteración", text="Iteración")
    tree.heading("Raíz Aproximada", text="Raíz Aproximada")
    tree.heading("f(Raíz)", text="f(Raíz)")
    tree.column("Iteración", anchor="center", width=100)
    tree.column("Raíz Aproximada", anchor="center", width=150)
    tree.column("f(Raíz)", anchor="center", width=150)

    # Insertar los resultados en la tabla
    for i, (c, fc) in enumerate(iterations):
        tree.insert("", "end", values=(i + 1, f"{c:.6f}", f"{fc:.6f}"))

    # Empaquetar el Treeview
    tree.pack(fill=tk.BOTH, expand=True)

root = tk.Tk()
root.title("Calculadora de Métodos Numéricos y Derivadas")

menu_bar = Menu(root)
root.config(menu=menu_bar)

funciones_menu = Menu(menu_bar, tearoff=0)
funciones_menu.add_command(label="f(x) = x^3 - x - 2", command=lambda: funcion_usuario.delete(0, tk.END) or funcion_usuario.insert(0, "x**3 - x - 2"))
funciones_menu.add_command(label="f(x) = sin(x)", command=lambda: funcion_usuario.delete(0, tk.END) or funcion_usuario.insert(0, "sin(x)"))
funciones_menu.add_command(label="f(x) = e^x - 3x^2", command=lambda: funcion_usuario.delete(0, tk.END) or funcion_usuario.insert(0, "exp(x) - 3*x**2"))
funciones_menu.add_command(label="f(x) = x^2 - 4", command=lambda: funcion_usuario.delete(0, tk.END) or funcion_usuario.insert(0, "x**2 - 4"))
menu_bar.add_cascade(label="Funciones Predefinidas", menu=funciones_menu)

frame_main = tk.Frame(root, bg="#edf0fa", bd=2, relief="groove")
frame_main.pack(padx=10, pady=10)

metodo_var = tk.StringVar(value="Bisección")

# Parte izquierda - Métodos
tk.Label(frame_main, text="MÉTODOS", bg="#edf0fa", font=("Arial", 10, "bold"), anchor="w").grid(row=0, column=0, sticky="w")
tk.Radiobutton(frame_main, text="Bisección", variable=metodo_var, value="Bisección", bg="#edf0fa", command=habilitar_entradas).grid(row=1, column=0, sticky="w")
tk.Radiobutton(frame_main, text="Falsa Posición", variable=metodo_var, value="Falsa Posición", bg="#edf0fa", command=habilitar_entradas).grid(row=2, column=0, sticky="w")
tk.Radiobutton(frame_main, text="Newton-Raphson", variable=metodo_var, value="Newton-Raphson", bg="#edf0fa", command=habilitar_entradas).grid(row=3, column=0, sticky="w")
tk.Radiobutton(frame_main, text="Secante", variable=metodo_var, value="Secante", bg="#edf0fa", command=habilitar_entradas).grid(row=4, column=0, sticky="w")

# Parte derecha - Entradas
labels = ["a", "b", "X0", "X1", "Tolerancia"]
entries = []
for i, label in enumerate(labels):
    tk.Label(frame_main, text=label, bg="#edf0fa").grid(row=i, column=1, padx=5)
    entry = tk.Entry(frame_main)
    entry.grid(row=i, column=2, padx=5)
    entries.append(entry)

entry_a, entry_b, entry_x0, entry_x1, entry_tol = entries

# Entrada función y botones
tk.Label(frame_main, text="Función", bg="#edf0fa").grid(row=5, column=0, padx=5, pady=(10, 0), sticky="e")
funcion_usuario = tk.Entry(frame_main, width=30)
funcion_usuario.grid(row=5, column=1, columnspan=2, pady=(10, 0))

boton_graficar = tk.Button(frame_main, text="Graficar Función", bg="#cde4ff", command=graficar_funcion)
boton_graficar.grid(row=6, column=1, pady=10, sticky="e")

boton_ejecutar = tk.Button(frame_main, text="Ejecutar Método", bg="#ccf4cc", command=ejecutar_metodo)
boton_ejecutar.grid(row=6, column=2, pady=10, sticky="w")

frame_resultados = tk.Frame(root)
frame_resultados.pack(pady=10)

habilitar_entradas()
root.mainloop()
