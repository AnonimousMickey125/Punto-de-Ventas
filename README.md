# Punto-de-Ventas
# Importación de la biblioteca tkinter para crear interfaces gráficas
import tkinter as tk
# Importación de messagebox para mostrar mensajes emergentes
from tkinter import messagebox
# Importación de ttkbootstrap para estilos personalizados
import ttkbootstrap as tb
# Importación de ttk para widgets avanzados
from tkinter import ttk
# Importación de la clase para manejar la conexión a la base de datos
from ConexionBaseDatos import ConexionBaseDatos
# Importación de clases para manejar diferentes módulos del sistema
from Proveedores import Proveedores
from Cliente import Clientes
from Productos import Productos
from Empleado import Empleados
from Ventas import Ventas
# Importación de hashlib para encriptar contraseñas
import hashlib

# Definición de la clase principal de la ventana, que hereda de tb.Window y ConexionBaseDatos
class Ventana(tb.Window, ConexionBaseDatos):
    def __init__(self):
        try:
            # Llamada al constructor de la clase base
            super().__init__()
            ConexionBaseDatos.__init__(self)  # Inicializa la conexión a la base de datos
            self.title("Sistema de la Papelería Tsurito")
            self.state("zoomed")
            tb.Style("superhero")
            self.rol_usuario = None
            self.frame_login = None
            self.frame_left = None
            self.frame_center = None
            self.frame_right = None
            self.ingresar_login()
        except Exception as e:
            messagebox.showerror("Error", f"Error al iniciar la aplicación: {str(e)}")

    # Método para encriptar contraseñas usando SHA-256
  def hashear_contraseña(self, contraseña):
    return hashlib.sha256(contraseña.encode()).hexdigest()

  def cerrar_sesion(self):
        try:
            if hasattr(self, 'frame_left') and self.frame_left is not None:
                self.frame_left.destroy()
                self.frame_left = None
            if hasattr(self, 'frame_center') and self.frame_center is not None:
                self.frame_center.destroy()
                self.frame_center = None
            if hasattr(self, 'frame_right') and self.frame_right is not None:
                self.frame_right.destroy()
                self.frame_right = None
            if hasattr(self, 'frame_login') and self.frame_login is not None:
                self.frame_login.destroy()
                self.frame_login = None
                self.rol_usuario = None
            self.ingresar_login()
        except Exception as e:
            messagebox.showerror("Error", f"Error al cerrar sesión: {str(e)}")

    # Método para mostrar el menú principal
  def ventana_menu(self):
        try:
            # Verifica si existe el frame de login y lo destruye
            if hasattr(self, 'frame_login') and self.frame_login is not None:
                self.frame_login.destroy()
                self.frame_login = None

            # Creación de los frames para organizar la interfaz
  self.frame_left = tk.Frame(self, width=200)
  self.frame_left.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")
  self.frame_center = tk.Frame(self)
  self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
  self.frame_right = tk.Frame(self, width=300)
  self.frame_right.grid(row=0, column=2, padx=10, pady=10, sticky="nsew")

            # Configuración de botones según el rol del usuario
  if self.rol_usuario == "admin":
                ttk.Button(self.frame_left, text="Productos", width=20, command=self.mostrar_productos).grid(row=1, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Proveedores", width=20, command=self.mostrar_proveedores).grid(row=2, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Empleados", width=20, command=self.mostrar_empleados).grid(row=3, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Clientes", width=20, command=self.mostrar_clientes).grid(row=4, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Ventas", width=20, command=self.mostrar_ventas).grid(row=5, column=0, padx=10, pady=10)
 elif self.rol_usuario == "empleado":
                ttk.Button(self.frame_left, text="Productos", width=20, command=self.mostrar_productos).grid(row=1, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Clientes", width=20, command=self.mostrar_clientes).grid(row=2, column=0, padx=10, pady=10)
                ttk.Button(self.frame_left, text="Ventas", width=20, command=self.mostrar_ventas).grid(row=3, column=0, padx=10, pady=10)

            # Botón para cerrar sesión
  ttk.Button(self.frame_left, text="Cerrar Sesión", width=20, command=self.cerrar_sesion).grid(row=6, column=0, padx=10, pady=10)
            # Etiqueta para el título del menú
        tk.Label(self.frame_center, text="Menú").grid(row=1, column=0, padx=10, pady=10)
        except Exception as e:
            messagebox.showerror("Error", f"Error al cargar el menú: {str(e)}")

    # Método para manejar el inicio de sesión
  def entrada(self):
        try:
            usuario = self.txt_usuario.get().strip()
            contraseña = self.txt_contraseña.get().strip()
            print(f"Intentando iniciar sesión con usuario: {usuario}, contraseña: {contraseña}")
            if not usuario or not contraseña:
                messagebox.showerror("Error", "Complete ambos campos")
                return

  conn = self.Obtener_conexion()
            cursor = conn.cursor()
            hashed_contraseña = self.hashear_contraseña(contraseña)
            print(f"Contraseña hasheada: {hashed_contraseña}")
            cursor.execute("SELECT rol FROM Usuarios WHERE nombre=? AND contraseña=?", (usuario, hashed_contraseña))
            resultado = cursor.fetchone()
            print(f"Resultado de la consulta: {resultado}")
            cursor.close()
            if resultado:
                self.rol_usuario = resultado[0]
                self.ventana_menu()
            else:
                messagebox.showerror("Error", "Usuario o contraseña incorrectos")
        except Exception as e:
            messagebox.showerror("Error", f"Error al iniciar sesión: {str(e)}")
            print(f"Excepción capturada: {str(e)}")

    # Métodos para mostrar las diferentes secciones del sistema
  def mostrar_proveedores(self):
        try:
            self.limpiar_centro()
            self.frame_center = Proveedores(self, self)
            self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        except Exception as e:
            messagebox.showerror("Error", f"Error al mostrar Proveedores: {str(e)}")

  def mostrar_clientes(self):
        try:
            self.limpiar_centro()
            self.frame_center = Clientes(self, self)
            self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        except Exception as e:
            messagebox.showerror("Error", f"Error al mostrar Clientes: {str(e)}")

  def mostrar_productos(self):
        try:
            self.limpiar_centro()
            self.frame_center = Productos(self, self)
            self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        except Exception as e:
            messagebox.showerror("Error", f"Error al mostrar Productos: {str(e)}")

  def mostrar_empleados(self):
        try:
            self.limpiar_centro()
            self.frame_center = Empleados(self, self)
            self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        except Exception as e:
            messagebox.showerror("Error", f"Error al mostrar Empleados: {str(e)}")

  def mostrar_ventas(self):
        try:
            self.limpiar_centro()
            self.frame_center = Ventas(self, self)
            self.frame_center.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        except Exception as e:
            messagebox.showerror("Error", f"Error al mostrar Ventas: {str(e)}")

    # Método para limpiar el contenido del frame central
  def limpiar_centro(self):
        try:
            for widget in self.frame_center.winfo_children():
                widget.destroy()
        except Exception as e:
            print(f"Error al limpiar centro: {str(e)}")

    # Método para mostrar la pantalla de login
  def ingresar_login(self):
        try:
     # Creación del frame para el login
            self.frame_login = tk.Frame(self)
            self.frame_login.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")

          # Creación de un LabelFrame para organizar los elementos del login
  self.lblframe_login = tk.LabelFrame(self.frame_login, text="Acceso")
  self.lblframe_login.grid(row=0, column=0, padx=10, pady=10)

            # Etiqueta para el título del login
  tk.Label(self.lblframe_login, text="Iniciar sesión").grid(row=0, column=0, padx=10, pady=10)

            # Campo de entrada para el usuario
  self.txt_usuario = ttk.Entry(self.lblframe_login, width=30, justify="center")
  self.txt_usuario.grid(row=1, column=0, padx=10, pady=10)
  self.txt_usuario.insert(0, "Usuario")
            # Limpiar el texto por defecto al hacer clic
  self.txt_usuario.bind("<FocusIn>", lambda event: self.txt_usuario.delete(0, tk.END) if self.txt_usuario.get() == "Usuario" else None)

            # Campo de entrada para la contraseña
  self.txt_contraseña = ttk.Entry(self.lblframe_login, width=30, show="*")
  self.txt_contraseña.grid(row=2, column=0, padx=10, pady=10)
  self.txt_contraseña.insert(0, "Contraseña")
            # Limpiar el texto por defecto al hacer clic
  self.txt_contraseña.bind("<FocusIn>", lambda event: self.txt_contraseña.delete(0, tk.END) if self.txt_contraseña.get() == "Contraseña" else None)

            # Etiqueta y combobox para seleccionar el rol del usuario
  tk.Label(self.lblframe_login, text="Selecciona tu rol:").grid(row=3, column=0, pady=(10, 0))
  self.combo_rol = ttk.Combobox(self.lblframe_login, width=28, state="readonly", justify="center")
     self.combo_rol['values'] = ("admin", "empleado")
     self.combo_rol.current(0)
      self.combo_rol.grid(row=4, column=0, padx=10, pady=10)

            # Botón para iniciar sesión
  ttk.Button(self.lblframe_login, text="Entrar", command=self.entrada).grid(row=5, column=0, padx=10, pady=10)
        except Exception as e:
            messagebox.showerror("Error", f"Error al cargar el login: {str(e)}")

# Función principal para iniciar la aplicación
def main():
    try:
        app = Ventana()
        app.mainloop()
    except Exception as e:
        print(f"Error al iniciar la aplicación: {str(e)}")

# Verifica si el archivo se ejecuta directamente
if __name__ == "__main__":
    main()
