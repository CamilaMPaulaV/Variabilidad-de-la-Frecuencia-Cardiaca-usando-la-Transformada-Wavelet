# Variabilidad de la Frecuencia Cardiaca usando la Transformada Wavelet
# Introducción
A continuación encontrará el código necesario apra medir la varibailidad de la frecuencia cardiaca (HRV) partiendo de una señal de electrocardiograma tomando con el sensor ECG AD8232 donde primeramente se mantuvo una frecuencia estabale en estado de reposo y calma para posteriormente alterarla con repiraciones repetitivas constantes a bajos intervalos.

Para la medición del HVR se hizo uso de la transformada Wavelet continua, se realizó el preprocesamiento de la señal para eliminar ruido, se ideintificaron los picos R, y se realizó el análisis tanto en el dominio del tiempo como en el dominio tiempo-frecuencia, posteriormente se identificó la dinámica en la señal cardíaca asociada a la actividad simpática y parasimpática del sistema nervioso autónomo, interpretando los resultados con base en los componentes espectrales de baja y alta frecuencia. 

# Resultados

# Instrucción

## Código para la adquisición de datos
1. Se prepara y configura la señal inicial, se limpia el entorno de trabajo, se cierran los puertos seriales previamente abiertos y se configura el nuevo puerto serial para la captura de datos.

```python
% 1. Preparación y configuración inicial
clc; clear all; close all;

% Cerrar puertos seriales abiertos previamente
if ~isempty(serialportlist)
    for p = serialportlist
        try clear(serialport(p)); catch, end
    end
end

% Configurar el puerto serial
puerto    = 'COM5';              % Puerto serial donde está conectado el dispositivo
baudios   = 115200;              % Velocidad de transmisión de datos
sp        = serialport(puerto, baudios); % Crear el objeto de puerto serial
sp.Timeout = 1;                  % Timeout corto para evitar bloqueos largos
flush(sp);                       % Limpiar cualquier dato previo en el puerto serial
```

2. Se abre el archivo PRUEBBA.txt donde se guardarán los datos del ECG
```python
    % 2. Apertura del archivo de registro
fname = 'PRUEBBA.txt';           % Nombre del archivo de salida
fid   = fopen(fname, 'w');       % Abrir el archivo en modo escritura
if fid == -1
    error('No se pudo abrir %s para escritura.', fname);  % Error si no se puede abrir
end
```

3. Se definen los parámetros de captura, como el tiempo total que son 5 minutos y los buffers para almacenar los datos crudos y los filtrados del ECG. También se prepara la figura para las gráficas.
```python
% 3. Inicialización de parámetros de captura y buffers
T_total    = 300;                % Duración total de la captura (5 minutos en segundos)
tStart     = tic;                % Iniciar el cronómetro
plotInterval = 0.1;              % Intervalo de actualización gráfica (0.1 segundos)
lastPlot   = tic;                % Variable para controlar el intervalo de gráficos

% Buffers para almacenar las muestras
bufSize    = 500;                % Tamaño del buffer de datos
ventanaMM  = 5;                  % Tamaño de la ventana para el filtro de media móvil
datos      = zeros(1, bufSize, 'uint8');  % Buffer para los datos crudos
datosF     = zeros(1, bufSize);           % Buffer para los datos filtrados

% Preparar la figura para las gráficas
hFig = figure('Name','ECG 5 min','NumberTitle','off');
hRaw  = plot(datos,'b','LineWidth',1);  % Gráfica para los datos crudos (en azul)
hold on;
hFilt = plot(datosF,'r','LineWidth',1); % Gráfica para los datos filtrados (en rojo)
ylim([0,255]);                        % Rango de valores para el ECG
grid on;                              % Habilitar la cuadrícula
xlabel('Muestras');                   % Etiqueta eje X
ylabel('Valor (uint8)');              % Etiqueta eje Y
hTitle = title('0 / 300 s','FontSize',12);  % Título de la gráfica

```
4. El bucle de captura se ejecuta durante 5 minutos. Lee los datos desde el puerto serial, los guarda en el archivo y actualiza las gráficas en tiempo real, además aplica un filtro de media móvil a los datos crudos.

```python
% 4. Bucle de captura y actualización gráfica
while toc(tStart) < T_total
    % Leer los datos disponibles desde el puerto serial
    nAvail = sp.NumBytesAvailable;
    if nAvail > 0
        chunk = read(sp, nAvail, 'uint8');  % Leer los datos disponibles
        % Guardar las muestras en el archivo
        fprintf(fid, '%u\n', chunk);

        % Actualizar los buffers de las muestras
        for y = chunk
            datos  = [datos(2:end), y];   % Añadir el nuevo dato al buffer de datos
        end
        % Aplicar filtro de media móvil a los datos crudos
        datosF = movmean(datos, ventanaMM);  % Filtrar los datos crudos
    end

    % Actualizar las gráficas cada plotInterval segundos
    if toc(lastPlot) > plotInterval
        set(hRaw,  'YData', datos);   % Actualizar la gráfica de los datos crudos
        set(hFilt, 'YData', datosF);  % Actualizar la gráfica de los datos filtrados
        elapsed = toc(tStart);        % Calcular el tiempo transcurrido
        set(hTitle,'String', sprintf('%.1f / 300 s', elapsed));  % Actualizar el título
        drawnow limitrate;            % Actualizar la gráfica de manera eficiente
        lastPlot = tic;               % Reiniciar el temporizador para la gráfica
    end
end

```
5. Al finalizar la captura de datos, se cierra el archivo donde se almacenaron los datos y se limpia el puerto serial.
```python
% 5. Cierre de archivo y reporte final
fclose(fid);             % Cerrar el archivo de registro
clear sp;                % Limpiar el objeto del puerto serial
fprintf('→ Captura completa de 5 minutos guardada en %s\n', fname);  % Mensaje de éxito
```

## Código de procesamiento de la señal

1. A continuación se carga la señal ECG desde un archivo de texto y definimos los parámetros de muestreo, asimismo se establece la frecuencia de muestreo (fs) y se calcula el tiempo correspondiente a cada muestra.

```python
# --- Cargar la señal ECG ---
signal_path = r"C:\\Users\\Camila Martinez\\Downloads\\PAULA001.txt"
signal_data = np.loadtxt(signal_path)

# Parámetros de muestreo
fs = 1000  # Hz
time = np.arange(len(signal_data)) / fs

```
2. Se diseña un filtro IIR Butterworth pasabanda para filtrar las frecuencias del ECG, se establecen los cortes de frecuencia y el orden del filtro (4), luego, visualizamos la respuesta en frecuencia del filtro.

```python
# --- Diseño del filtro IIR Butterworth pasabanda ---
lowcut = 0.5  # Hz
highcut = 40.0  # Hz
order = 4

# Filtro en forma directa y SOS
b, a = signal.butter(order, [lowcut, highcut], btype='bandpass', fs=fs)
sos = signal.butter(order, [lowcut, highcut], btype='bandpass', fs=fs, output='sos')

# Visualizar respuesta en frecuencia del filtro (forma SOS)
w, h = signal.sosfreqz(sos, worN=2048, fs=fs)
magnitude = 20 * np.log10(np.maximum(np.abs(h), 1e-10))

plt.figure(figsize=(10, 5))
plt.semilogx(w, magnitude, label='Ganancia (dB)')
plt.axvline(lowcut, color='red', linestyle='--', label=f'Corte baja: {lowcut} Hz')
plt.axvline(highcut, color='green', linestyle='--', label=f'Corte alta: {highcut} Hz')
plt.title("Respuesta en Frecuencia del Filtro IIR Butterworth (Pasabanda)")
plt.xlabel("Frecuencia (Hz)")
plt.ylabel("Magnitud (dB)")
plt.ylim([-60, 5])
plt.grid(True, which='both')
plt.legend()
plt.tight_layout()
plt.show()
```

3. Se imprimen los coeficientes del filtro (numerador y denominador), y se muestra la ecuación en diferencias asociada al filtro IIR.
```python
# Mostrar ecuación en diferencias (coeficientes)
print("Coeficientes b:", b)
print("Coeficientes a:", a)
print("\nEcuación en diferencias:")
print("y[n] = " + " + ".join([f"{b[i]:.4f}*x[n-{i}]" for i in range(len(b))]) +
      " - " + " - ".join([f"{a[j]:.4f}*y[n-{j}]" for j in range(1, len(a))]))
```

4. Se impplementa de manera manualmente el filtro utilizando las ecuaciones en diferencias y luego aplicamos el filtro a la señal ECG.
```python
# Implementación manual del filtro (condiciones iniciales en 0)
def apply_iir_filter(x, b, a):
    y = np.zeros_like(x)
    for n in range(len(x)):
        for i in range(len(b)):
            if n - i >= 0:
                y[n] += b[i] * x[n - i]
        for j in range(1, len(a)):
            if n - j >= 0:
                y[n] -= a[j] * y[n - j]
    return y

filtered = apply_iir_filter(signal_data, b, a)

```
5. Se detectan los picos R en la señal ECG filtrada, calculamos los intervalos RR y los convertimos a milisegundos.

```python
# --- Detección de picos R ---
distance = int(0.6 * fs)  # Asegura una separación mínima entre los picos R
peaks, _ = signal.find_peaks(filtered, distance=distance, height=np.mean(filtered))

# --- Cálculo de intervalos R-R ---
rr_intervals = np.diff(peaks) / fs * 1000  # en milisegundos

```

6. Se calculan varios parámetros de variabilidad de la frecuencia cardíaca (HRV), como la media, SDNN, RMSSD y pNN50, basados en los intervalos RR.
   
```python
# --- Parámetros HRV (dominio del tiempo) ---
mean_rr = np.mean(rr_intervals)
sdnn = np.std(rr_intervals)
rmssd = np.sqrt(np.mean(np.square(np.diff(rr_intervals))))
nn50 = np.sum(np.abs(np.diff(rr_intervals)) > 50)
pnn50 = nn50 / len(rr_intervals) * 100

```
7. Se aplica una transformada wavelet continua (CWT) de los intervalos RR para obtener una representación espectral de la variabilidad de la frecuencia cardíaca.
```python
# --- Transformada Wavelet Continua (CWT) ---
scales = np.arange(1, 128)
cwt_matrix, _ = pywt.cwt(rr_intervals, scales, 'morl')
freqs = pywt.scale2frequency('morl', scales) * fs * 1000 / np.mean(np.diff(peaks))

```
8. Se realizan tres gráficas, la primera con la señal filtrada con los picos R,  la segunda los intervalos RR y la tercera el espectrograma de la transformada wavelet.

```python
# --- Gráficas ---
plt.figure(figsize=(18, 12))

# Señal filtrada y picos R
plt.subplot(3, 1, 1)
plt.plot(time, filtered, label="ECG filtrada", linewidth=1)
plt.plot(peaks / fs, filtered[peaks], 'r.', label="Picos R")
plt.title("Señal ECG filtrada y detección de picos R")
plt.xlabel("Tiempo (s)")
plt.ylabel("Amplitud")
plt.legend()
plt.grid(True)

# Intervalos RR
plt.subplot(3, 1, 2)
plt.plot(rr_intervals, label="RR Intervalos (ms)")
plt.axhline(mean_rr, color='r', linestyle='--', label=f"Media RR = {mean_rr:.2f} ms")
plt.title("Parámetros HRV - Dominio del Tiempo")
plt.xlabel("Latido")
plt.ylabel("Intervalo RR (ms)")
plt.legend()
plt.grid(True)

# Espectrograma Wavelet
plt.subplot(3, 1, 3)
plt.imshow(np.abs(cwt_matrix), extent=[0, len(rr_intervals), freqs[-1], freqs[0]],
           aspect='auto', cmap='jet')
plt.colorbar(label="Potencia")
plt.axhline(0.04, color='magenta', linestyle='--', linewidth=1.5, label='Inicio LF (0.04 Hz)')
plt.axhline(0.15, color='white', linestyle='--', linewidth=1.5, label='Límite LF/HF (0.15 Hz)')
plt.axhline(0.4, color='cyan', linestyle='--', linewidth=1.5, label='Límite HF (0.4 Hz)')
plt.title("Transformada Wavelet Continua - HRV")
plt.xlabel("Latido")
plt.ylabel("Frecuencia (Hz)")
plt.legend()
plt.grid(False)

plt.tight_layout()
plt.show()

```

9. Se imprimen los resultados de HRV calculados y algunas observaciones relacionadas con la fisiología de la variabilidad de la frecuencia cardíaca.
    
```python
# --- Resultados HRV ---
print("\nRESULTADOS HRV")
print(f"Media RR: {mean_rr:.2f} ms")
print(f"SDNN: {sdnn:.2f} ms")
print(f"RMSSD: {rmssd:.2f} ms")
print(f"pNN50: {pnn50:.2f} %")
print(f"Latidos detectados: {len(peaks)}")

# Observaciones fisiológicas
print("\nObservaciones:")
print(" LF (0.04–0.15 Hz): refleja actividad simpática y parasimpática.")
print(" HF (0.15–0.4 Hz): indica tono vagal relacionado con la respiración.")
print(" Se observa cómo la potencia varía a lo largo del tiempo en estas bandas.")
print(" Picos de HF podrían asociarse a estrés o respiración rápida.")
print(" Aumento de LF podría indicar predominancia simpática.")

```

## Uso
Statistical analysis of a signal by Camila Martínez and Paula Vega  
Published 30/04/25

## Referencias
