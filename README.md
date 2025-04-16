import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import os

# Crear ventana principal
ventana = tk.Tk()
ventana.title("Avicontrol Suite - Registro de Producción Avícola")
ventana.geometry("900x600")
ventana.configure(bg='#fdf6e3')

# Estilos
estilo_label = {'font': ('Arial', 12), 'bg': '#fdf6e3'}
estilo_entry = {'font': ('Arial', 12)}

# Datos
tipos_huevo = ["Tipo-A", "Tipo-AA", "Tipo-B", "Tipo-C", "Tipo-Industrial"]

campos = {}

# Función para generar reporte
def generar_reporte():
    datos = {}
    for tipo in tipos_huevo:
        try:
            anterior = int(campos[f"Anterior-{tipo}"].get())
            producido = int(campos[f"Producido-{tipo}"].get())
            salida = int(campos[f"Salida-{tipo}"].get())
            inventario = anterior + producido - salida

            datos[tipo] = {
                "Anterior": anterior,
                "Producido": producido,
                "Salida": salida,
                "Inventario": inventario
            }
        except ValueError:
            messagebox.showerror("Error", f"Por favor ingrese todos los valores numéricos para {tipo}")
            return

    # Crear DataFrame
    df = pd.DataFrame(datos).T
    df.index.name = "Tipo de Huevo"

    # Guardar Excel
    hoy = datetime.datetime.now().strftime('%Y-%m-%d')
    nombre_excel = f"produccion_{hoy}.xlsx"
    df.to_excel(nombre_excel)

    # Generar gráfico y guardar
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.bar(df.index, df["Producido"], color=['#f7e1b5', '#f5c16c', '#deb887', '#d2a679', '#a0522d'])
    ax.set_title('Producción del Día por Tipo de Huevo')
    ax.set_xlabel('Tipo de Huevo')
    ax.set_ylabel('Cantidad Producida')
    for i, cantidad in enumerate(df["Producido"]):
        ax.text(i, cantidad + 2, str(cantidad), ha='center')
    plt.tight_layout()
    nombre_img = f"produccion_{hoy}.png"
    plt.savefig(nombre_img)
    plt.close()

    messagebox.showinfo("Éxito", f"Reporte generado:\n{nombre_excel}\n{nombre_img}")

# Función para mostrar gráficos
def ver_graficos():
    hoy = datetime.datetime.now().strftime('%Y-%m-%d')
    nombre_img = f"produccion_{hoy}.png"
    if os.path.exists(nombre_img):
        img = plt.imread(nombre_img)
        plt.imshow(img)
        plt.axis('off')
        plt.title("Gráfico de Producción")
        plt.show()
    else:
        messagebox.showwarning("Aviso", "Primero debe generar el reporte.")

# Crear interfaz gráfica
fila = 1
tk.Label(ventana, text="Nombre del Cliente:", **estilo_label).grid(row=0, column=0, sticky='e')
cliente_entry = tk.Entry(ventana, **estilo_entry)
cliente_entry.grid(row=0, column=1, columnspan=2, pady=5, sticky='we')

for tipo in tipos_huevo:
    tk.Label(ventana, text=f"Producción del día anterior ({tipo}):", **estilo_label).grid(row=fila, column=0, sticky='e')
    campos[f"Anterior-{tipo}"] = tk.Entry(ventana, **estilo_entry)
    campos[f"Anterior-{tipo}"].grid(row=fila, column=1, pady=2)

    tk.Label(ventana, text=f"Producción del día ({tipo}):", **estilo_label).grid(row=fila, column=2, sticky='e')
    campos[f"Producido-{tipo}"] = tk.Entry(ventana, **estilo_entry)
    campos[f"Producido-{tipo}"].grid(row=fila, column=3, pady=2)

    tk.Label(ventana, text=f"Salida ({tipo}):", **estilo_label).grid(row=fila, column=4, sticky='e')
    campos[f"Salida-{tipo}"] = tk.Entry(ventana, **estilo_entry)
    campos[f"Salida-{tipo}"].grid(row=fila, column=5, pady=2)

    fila += 1

# Botones
btn_generar = tk.Button(ventana, text="Generar Reporte", command=generar_reporte, bg='#a0522d', fg='white', font=('Arial', 12, 'bold'))
btn_generar.grid(row=fila + 1, column=1, pady=20)

btn_ver = tk.Button(ventana, text="Ver Gráficos de Producción", command=ver_graficos, bg='#deb887', font=('Arial', 12))
btn_ver.grid(row=fila + 1, column=3, pady=20)

ventana.mainloop()
