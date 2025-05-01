# Variabilidad de la Frecuencia Cardiaca usando la Transformada Wavelet
# Introducción
A continuación encontrará el código necesario apra medir la varibailidad de la frecuencia cardiaca (HRV) partiendo de una señal de electrocardiograma tomando con el sensor ECG AD8232 donde primeramente se mantuvo una frecuencia estabale en estado de reposo y calma para posteriormente alterarla con repiraciones repetitivas constantes a bajos intervalos.

Para la medición del HVR se hizo uso de la transformada Wavelet continua, se realizó el preprocesamiento de la señal para eliminar ruido, se ideintificaron los picos R, y se realizó el análisis tanto en el dominio del tiempo como en el dominio tiempo-frecuencia, posteriormente se identificó la dinámica en la señal cardíaca asociada a la actividad simpática y parasimpática del sistema nervioso autónomo, interpretando los resultados con base en los componentes espectrales de baja y alta frecuencia. 

# Fundamento Teórico

## Sistema Nervioso Autónomo (SNA)
El sistema nervioso autónomo regula funciones involuntarias del cuerpo, incluyendo la frecuencia cardiaca. Está compuesto por dos ramas principales:

- Simpática: Aumenta la frecuencia cardiaca y la contractilidad del corazón, preparándolo para situaciones de estrés o actividad (“respuesta de lucha o huida”).
- Parasimpática: Disminuye la frecuencia cardiaca mediante el nervio vago, favoreciendo el estado de reposo y recuperación (“respuesta de descanso y digestión”).

## Variabilidad de la Frecuencia Cardiaca (HRV)
La HRV representa las fluctuaciones en el tiempo entre latidos sucesivos del corazón (intervalos R-R). Es un biomarcador del control autónomo del corazón y se asocia con estados de salud física y mental. Las métricas típicas incluyen:

- Media de los intervalos R-R

- Desviación estándar (SDNN)

- RMSSD (Root Mean Square of Successive Differences)

- pNN50 (% de pares de R-R con diferencia > 50 ms)

Las componentes de frecuencia relevantes para el análisis de HRV son:

- Baja frecuencia (LF, 0.04–0.15 Hz): Relacionada con regulación simpática y parasimpática.

- Alta frecuencia (HF, 0.15–0.4 Hz): Asociada principalmente a la actividad parasimpática.

## Transformada Wavelet
La transformada wavelet permite el análisis tiempo-frecuencia de señales no estacionarias como el ECG. A diferencia de la transformada de Fourier, ofrece resolución temporal alta para altas frecuencias y resolución en frecuencia para las bajas.

Tipos de Wavelets
- Haar: simple pero poco suave, útil para cambios bruscos.

- Daubechies: más suave y apropiada para señales fisiológicas como ECG.

- Symlets y Coiflets: también usadas en bioseñales por su buena localización.

La transformada wavelet puede ser:

- Continua (CWT): proporciona redundancia y permite un análisis más detallado.

- Discreta (DWT): más eficiente computacionalmente y suficiente para análisis de HRV.

# Diagrama de flujo

![image](https://github.com/user-attachments/assets/54589cf3-9177-4621-adf8-9d11c475a009)


# Resultados

## Respuesta en frecuencia del filtro IIR
![image](https://github.com/user-attachments/assets/926c7545-7ffc-4fc8-9072-1ca379b681ec)

En esta imagen se observa la respuesta en frecuencia de un filtro IIR Butterworth pasabanda. Este tipo de filtro se caracteriza por presentar una respuesta suave y sin ondulaciones en la banda pasante, ideal para señales biomédicas, como lo es el caso de la señal ECG. El filtro diseñado permite el paso de frecuencias entre 0.5 Hz y 40 Hz, debido a que estas permiten eliminar el ruido generado por movimientos, fluctuaciones respiratorias o interferencia electromagnética. La ganancia dentro de la banda pasante es cercana a 0 dB, por lo cual las componentes útiles de la señal no se ven afectadas significativamente. Las pendientes pronunciadas fuera de la banda muestran una buena atenuación.

## Análisis HRV
![image](https://github.com/user-attachments/assets/60001634-f4af-4922-b0f7-8c5ff268ac97)

En el primer gráfico se muestra la señal ECG filtrada, sobre la cual se destacan los picos R detectados que corresponden a los máximos del complejo QRS. La detección de estos picos es fundamental, ya que permiten calcular los intervalos RR, es decir, el tiempo que existe entre cada latido.

En segundo gráfico representa los intervalos RR en el dominio del tiempo, mostrando su variabilidad latido a latido. La línea azul muestra cómo varía el tiempo entre latidos, mientras que la línea roja discontinua representa la media de los intervalos RR, que en este caso es de aproximadamente 844.65 ms. Esto sugiere una frecuencia cardíaca promedio cercana a 71 latidos por minuto, considerándose así normal. 

Por último, el tercer gráfico presenta un análisis en el dominio del tiempo-frecuencia usando la transformada wavelet continua (CWT) aplicada a la señal HRV. Este gráfico permite observar cómo evoluciona la potencia espectral del HRV a lo largo de los latidos, se identifica claramente las bandas de interés fisiológico: la banda LF (Low Frequency, 0.04–0.15 Hz) asociada a actividad simpática y parasimpática, y la banda HF (High Frequency, 0.15–0.4 Hz) relacionada principalmente con la modulación parasimpática. El mapa de colores indica la magnitud de la potencia en cada frecuencia y momento, donde los colores cálidos indican mayor actividad. También se observa que hay modulación significativa en ambas bandas, sugiriendo una adecuada función autonóma.

![image](https://github.com/user-attachments/assets/f69dc40f-2aac-452f-899c-2b82722ee47a)

Los coeficientes proporcionados permiten expresar el filtro mediante su ecuación en diferencias, que describe cómo se calcula la salida actual 𝑦[𝑛] del sistema en función de entradas anteriores 
𝑥[𝑛−𝑘] y salidas anteriores 𝑦[𝑛−𝑘]. La ecuación en diferencias obtenida muestra una combinación de coeficientes simétricos y ceros intercalados en los términos del numerador 𝑏, lo que contribuye a una respuesta más precisa y estable en la banda pasante. Por otro lado, los coeficientes del denominador 𝑎 indican una retroalimentación fuerte. Esta estructura proporciona un comportamiento eficiente para el procesamiento de señales biológicas para  conservar la morfología de la señal útil y eliminar el ruido.

Los resultados obtenidos indican una media de los intervalos RR de 844.65 milisegundos, lo cual equivale a una frecuencia cardíaca aproximada de 71 latidos por minuto. Este valor se encuentra dentro del rango normal para una persona en estado de reposo.

La desviación estándar de los intervalos RR, es de 185.09 ms. Este valor representa la variabilidad general del ritmo cardíaco durante el periodo analizado. Valores superiores a 100 ms suelen considerarse positivos.

El RMSSD (raíz cuadrada de la media de las diferencias cuadráticas sucesivas entre intervalos RR) tiene un valor de 246.35 ms, que es notablemente alto. Este parámetro está asociado con la actividad del sistema parasimpático, el cual predomina en situaciones de reposo y relajación.

El valor de pNN50, que indica el porcentaje de intervalos RR sucesivos que difieren en más de 50 ms, es de 79.58 %. Un resultado elevado, que se encuentra relacionado con individuos sanos.


# Instrucción

## Código para la adquisición de datos
1. Se prepara y configura la señal inicial, se limpia el entorno de trabajo, se cierran los puertos seriales previamente abiertos y se configura el nuevo puerto serial para la captura de datos.

```python
clc; clear all; close all;

% Cerrar puertos seriales abiertos previamente
if ~isempty(serialportlist)
    for p = serialportlist
        try clear(serialport(p)); catch, end
    end
end

% Configurar el puerto serial
puerto    = 'COM5';              
baudios   = 115200;              
sp        = serialport(puerto, baudios); 
sp.Timeout = 1;                  
flush(sp);                       
```

2. Se abre el archivo PRUEBBA.txt donde se guardarán los datos del ECG
```python

fname = 'PRUEBBA.txt';       
fid   = fopen(fname, 'w');      
if fid == -1
    error('No se pudo abrir %s para escritura.', fname);  
end
```

3. Se definen los parámetros de captura, como el tiempo total que son 5 minutos y los buffers para almacenar los datos crudos y los filtrados del ECG. También se prepara la figura para las gráficas.
```python

T_total    = 300;          
tStart     = tic;               
plotInterval = 0.1;              
lastPlot   = tic;                

bufSize    = 500;             
ventanaMM  = 5;                 
datos      = zeros(1, bufSize, 'uint8'); 
datosF     = zeros(1, bufSize);           

hFig = figure('Name','ECG 5 min','NumberTitle','off');
hRaw  = plot(datos,'b','LineWidth',1);  
hold on;
hFilt = plot(datosF,'r','LineWidth',1); 
ylim([0,255]);                       
grid on;                              
xlabel('Muestras');                   
ylabel('Valor (uint8)');              
hTitle = title('0 / 300 s','FontSize',12);  

```
4. El bucle de captura se ejecuta durante 5 minutos. Lee los datos desde el puerto serial, los guarda en el archivo y actualiza las gráficas en tiempo real, además aplica un filtro de media móvil a los datos crudos.

```python

while toc(tStart) < T_total
    % Leer los datos disponibles desde el puerto serial
    nAvail = sp.NumBytesAvailable;
    if nAvail > 0
        chunk = read(sp, nAvail, 'uint8');  
        fprintf(fid, '%u\n', chunk);

        for y = chunk
            datos  = [datos(2:end), y];   
        end
       
        datosF = movmean(datos, ventanaMM);  
    end

    % Actualizar las gráficas cada plotInterval segundos
    if toc(lastPlot) > plotInterval
        set(hRaw,  'YData', datos);   
        set(hFilt, 'YData', datosF);  
        elapsed = toc(tStart);        
        set(hTitle,'String', sprintf('%.1f / 300 s', elapsed));  
        drawnow limitrate;            
        lastPlot = tic;               
    end
end

```
5. Al finalizar la captura de datos, se cierra el archivo donde se almacenaron los datos y se limpia el puerto serial.
```python

fclose(fid);             
clear sp;                
fprintf('→ Captura completa de 5 minutos guardada en %s\n', fname);  
```

## Código de procesamiento de la señal

1. A continuación se carga la señal ECG desde un archivo de texto y definimos los parámetros de muestreo, asimismo se establece la frecuencia de muestreo (fs) y se calcula el tiempo correspondiente a cada muestra.

```python

signal_path = r"C:\\Users\\Camila Martinez\\Downloads\\PAULA001.txt"
signal_data = np.loadtxt(signal_path)

fs = 1000  
time = np.arange(len(signal_data)) / fs

```
2. Se diseña un filtro IIR Butterworth pasabanda para filtrar las frecuencias del ECG, se establecen los cortes de frecuencia y el orden del filtro (4), luego, visualizamos la respuesta en frecuencia del filtro.

```python

lowcut = 0.5  
highcut = 40.0  
order = 4

b, a = signal.butter(order, [lowcut, highcut], btype='bandpass', fs=fs)
sos = signal.butter(order, [lowcut, highcut], btype='bandpass', fs=fs, output='sos')

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

print("Coeficientes b:", b)
print("Coeficientes a:", a)
print("\nEcuación en diferencias:")
print("y[n] = " + " + ".join([f"{b[i]:.4f}*x[n-{i}]" for i in range(len(b))]) +
      " - " + " - ".join([f"{a[j]:.4f}*y[n-{j}]" for j in range(1, len(a))]))
```

4. Se impplementa de manera manualmente el filtro utilizando las ecuaciones en diferencias y luego aplicamos el filtro a la señal ECG.
```python

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

distance = int(0.6 * fs)  # Asegura una separación mínima entre los picos R
peaks, _ = signal.find_peaks(filtered, distance=distance, height=np.mean(filtered))

rr_intervals = np.diff(peaks) / fs * 1000  # en milisegundos

```

6. Se calculan varios parámetros de variabilidad de la frecuencia cardíaca (HRV), como la media, SDNN, RMSSD y pNN50, basados en los intervalos RR.
   
```python

mean_rr = np.mean(rr_intervals)
sdnn = np.std(rr_intervals)
rmssd = np.sqrt(np.mean(np.square(np.diff(rr_intervals))))
nn50 = np.sum(np.abs(np.diff(rr_intervals)) > 50)
pnn50 = nn50 / len(rr_intervals) * 100

```
7. Se aplica una transformada wavelet continua (CWT) de los intervalos RR para obtener una representación espectral de la variabilidad de la frecuencia cardíaca.
```python

scales = np.arange(1, 128)
cwt_matrix, _ = pywt.cwt(rr_intervals, scales, 'morl')
freqs = pywt.scale2frequency('morl', scales) * fs * 1000 / np.mean(np.diff(peaks))

```
8. Se realizan tres gráficas, la primera con la señal filtrada con los picos R,  la segunda los intervalos RR y la tercera el espectrograma de la transformada wavelet.

```python

plt.figure(figsize=(18, 12))


plt.subplot(3, 1, 1)
plt.plot(time, filtered, label="ECG filtrada", linewidth=1)
plt.plot(peaks / fs, filtered[peaks], 'r.', label="Picos R")
plt.title("Señal ECG filtrada y detección de picos R")
plt.xlabel("Tiempo (s)")
plt.ylabel("Amplitud")
plt.legend()
plt.grid(True)


plt.subplot(3, 1, 2)
plt.plot(rr_intervals, label="RR Intervalos (ms)")
plt.axhline(mean_rr, color='r', linestyle='--', label=f"Media RR = {mean_rr:.2f} ms")
plt.title("Parámetros HRV - Dominio del Tiempo")
plt.xlabel("Latido")
plt.ylabel("Intervalo RR (ms)")
plt.legend()
plt.grid(True)


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
print(" Picos de HF podrían asociarse a respiración rápida.")
print(" Aumento de LF indicando predominancia simpática.")

```

## Uso
Statistical analysis of a signal by Camila Martínez and Paula Vega  
Published 30/04/25

## Referencias
[1] MSD Manual, “Introducción al sistema nervioso autónomo,” MSD Manual Versión para público general. [En línea]. Disponible en: https://www.msdmanuals.com/es/hogar/enfermedades-cerebrales-medulares-y-nerviosas/trastornos-del-sistema-nervioso-aut%C3%B3nomo/introducci%C3%B3n-al-sistema-nervioso-aut%C3%B3nomo. [Accedido: 30-abr-2025].
