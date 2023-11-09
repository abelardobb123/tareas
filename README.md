# tareas
import serial
import pandas as pd
import random
import datetime
import mysql.connector
import time

# Constantes específicas de tu sensor de temperatura
pendiente_temperatura = 104.0
temperatura_minima = -20.0
frecuencia_maxima = 1000

# Constantes específicas de tu sensor de vibración
pendiente_vibracion = 5.0
interseccion_vibracion = 0.0

valor_maximo = 1023
voltaje_maximo = 5.0

ser = serial.Serial('/dev/ttyUSB0', 9600)
ser.flushInput()

# Conectar a la base de datos
conexion = mysql.connector.connect(
    host='localhost',
    user='root',
    password='',
    database='senatech'
)
cursor = conexion.cursor()

# Función para obtener la clasificación del estado del motor
def obtener_estado(temperatura, vibracion):
    if temperatura >= 70 or vibracion >= 0.8:
        return 'muy malo'
    elif temperatura < 40 and vibración < 0.4:
        return 'regular'
    elif 40 <= temperatura < 70 or 0.4 <= vibración < 0.8:
        return 'malo'
    else:
        return None

# Función para leer datos desde el archivo CSV
def leer_datos_desde_csv():
    try:
        # Leer el archivo CSV y almacenar los datos en un DataFrame
        df = pd.read_csv('datosMotorSeleccionado.csv')

        # Obtener los datos de la primera fila
        motor = df['Motor'][0]

        return motor

    except FileNotFoundError:
        print("El archivo 'datosMotorSeleccionado.csv' no se encontró.")
        return None

# Ejemplo de uso de la función
datos = leer_datos_desde_csv()

motor_temp = 30.0
numero_aleatorio = 0.0
motor_vib = 0.0

# Asegúrate de que capmot_serie no sea nulo
if datos is not None:
    capmot_serie = datos  # Asignar el valor de datos a capmot_serie
else:
    capmot_serie = 'ValorPredeterminado'  # Asignar un valor predeterminado en caso de que datos sea nulo

while True:
    try:
        datos_temperatura = ser.readline().decode('utf-8').strip()
        datos_vibracion = ser.readline().decode('utf-8').strip()

        # Obtener los valores de temperatura y vibración
        voltaje_temperatura = (voltaje_maximo / valor_maximo) * float(datos_temperatura)
        frecuencia_vibracion = float(datos_vibracion) * (frecuencia_maxima / valor_maximo)

        # Convierte el valor del voltaje a temperatura
        temperatura = pendiente_temperatura * voltaje_temperatura + temperatura_minima

        print(f"Datos enviados por el Arduino para la temperatura: {datos_temperatura}")
        print(f"Voltaje Temperatura: {voltaje_temperatura} V")
        print(f"Temperatura: {temperatura}°C")
        print("___________________________________________________________")
        print("")

        print(f"Datos enviados por el Arduino para la vibración : {datos_vibracion}")
        print(f"Frecuencia vibración: {frecuencia_vibracion} Hz")
        print("___________________________________________________________")
        print("")

        # Insertar datos en la base de datos
        query = "INSERT INTO motor_captura (capmot_serie, capmot_temperatura, capmot_vibracion, capmot_fecha, capmot_hora) VALUES (%s, %s, %s, %s, %s)"
        valores = (capmot_serie, temperatura, frecuencia_vibracion, datetime.datetime.now().strftime('%Y-%m-%d'), datetime.datetime.now().strftime('%H:%M:%S'))

        cursor.execute(query, valores)
        conexion.commit()

        time.sleep(60)  # Pausa de 60 segundos entre las inserciones de datos

    except KeyboardInterrupt:
        break

# Confirmar y cerrar la conexión a la base de datos
conexion.commit()
conexion.close()

import serial

# Constantes específicas de tu sensor de temperatura
pendiente_temperatura = 104.0  # Ree

#remplaza con la pendiente correcta
temperatura_minima = -20.0  # Reemplaza con la intersección correcta
frecuencia_maxima = 1000

# Constantes específicas de tu sensor de vibración
pendiente_vibracion = 5.0  # Reemplaza con la pendiente correcta
interseccion_vibracion = 0.0  # Reemplaza con la intersección correcta

#
valor_maximo = 1023
voltaje_maximo = 5.0

ser = serial.Serial('/dev/ttyUSB0', 9600)
ser.flushInput()

while True:
    try:
        datos_temperatura = ser.readline().decode('utf-8').strip()
        
        datos_vibracion = ser.readline().decode('utf-8').strip()
        
        voltaje_temperatura = (voltaje_maximo / valor_maximo) * float(datos_temperatura)
        frecuencia_vibracion = float(datos_vibracion) * (frecuencia_maxima / valor_maximo)
        
        # Convierte el valor del voltaje a temperatura
        temperatura = pendiente_temperatura * voltaje_temperatura + temperatura_minima
        #temperatura = voltaje_maximo / valor_maximo * float(datos_temperatura) * 100 - temperatura_minima
    
        
        print(f"Datos enviados por el Arduino para la temperatura: {datos_temperatura}")
        print(f"Voltaje Temperatura: {voltaje_temperatura} V")
        print(f"Temperatura: {temperatura}°C")
        print("_____________________")
        print("")
        
        print(f"Datos enviados por el Arduino para la vibración : {datos_vibracion}")
        print(f"Frecuencia vibracion: {frecuencia_vibracion} Hz")
        #print(f"vibracion: {vibracion}")
        #print(f"Voltaje Vibración: {voltaje_vibracion} V, Vibración: {vibracion} unidades")
        print("_____________________")
        print("")
        
    except KeyboardInterrupt:
        break

ser.close()
