# PROYECTO-IA-FINAL
Aqui se adjuntan todos los documentos y archivos utilizados para el proyecto.
# Alerta temprana de PM10 en Lima Metropolitana con una CNN-1D (SENAMHI 2015-2024)

Proyecto final del curso de Inteligencia Artificial. Autor: **Nicolas Alexander Torres Yucra** (FIEI – Ing. Informática).

## 1. ¿Cuál es el problema?
Entre 2015 y 2024, 1 913 días-estación de Lima Metropolitana tuvieron una media de PM10 mayor que el Estándar de Calidad Ambiental (ECA) de 24 horas (100 µg/m³, D.S. N.° 003-2017-MINAM). En 32 de 68 años-estación se superaron las 7 excedencias anuales permitidas. Esas excedencias se conocen cuando ya ocurrieron: no hay un aviso previo por estación.

## 2. ¿Cuál es el objetivo?
Desarrollar una **red neuronal convolucional 1D (CNN-1D)** que, con las últimas 72 horas de PM10, PM2.5 y NO2 de una estación, estime la probabilidad de que **mañana** la media de PM10 supere 100 µg/m³. El modelo se integra en un sistema de alerta temprana con semáforo.

## 3. ¿Qué dataset se utilizó?
*Monitoreo de los contaminantes del aire en Lima Metropolitana – [SENAMHI]*, publicado en la Plataforma Nacional de Datos Abiertos con licencia ODC-BY. Son datos horarios validados de 7 estaciones, del 01/01/2015 al 31/05/2024: 577 794 filas y 15 columnas.

## 4. ¿Cómo obtener los datos?
1. Descarga el recurso CSV desde https://www.datosabiertos.gob.pe/dataset/monitoreo-de-los-contaminantes-del-aire-en-lima-metropolitana-servicio-nacional-de
2. Guárdalo en `data/raw/`. En Colab, guárdalo en `MyDrive/PROYECTO-IA-FINAL/`.
3. El notebook verifica la versión con SHA-256. El valor esperado es `896ae8a106fb128460e81b3f58096f6b9c4d582ef28fffa46e5394c5875c40d4` (68 501 558 bytes).

## 5. ¿Cómo instalar el proyecto?
```bash
pip install -r requirements.txt
```
Requiere Python 3.10 o superior. No necesita GPU: la CNN está programada en NumPy.

## 6. ¿Cómo ejecutar el código?
Abre `notebooks/PROYECTO_IA_FINAL_(TORRES).ipynb` en Google Colab, Jupyter o VS Code y ejecuta todas las celdas. El notebook crea `src/`, `tests/`, `prototipo/`, `modelos/` y `resultados/`.

## 7. ¿Cómo entrenar el modelo?
El notebook entrena los experimentos E3 (MLP), E4 (CNN-1D) y E5 (CNN-1D solo con PM10) con 5 semillas cada uno. Tarda unos 8 minutos en CPU. El modelo final se exporta a `modelos/cnn1d_alerta_pm10.npz` y `.json`.

## 8. ¿Cómo reproducir los experimentos?
Usa el mismo CSV (el notebook comprueba el SHA-256), las semillas `[42, 7, 123, 2024, 99]` y la misma división temporal:
- entrenamiento: 2015-2019
- validación: 2020-2021
- prueba: 2022-2024, con embargo entre conjuntos

Las pruebas unitarias se ejecutan con:
```bash
python tests/test_proyecto.py
```

## 9. ¿Cómo ejecutar el prototipo?
```bash
python prototipo/alerta_pm10.py --csv "data/raw/<archivo SENAMHI>.csv" --fecha 2022-05-23
```
Para cada estación imprime la probabilidad de excedencia y el nivel **ROJO / AMARILLO / VERDE**, y guarda el boletín en `resultados/boletin_alerta_<fecha>.csv`.

## 10. ¿Cuáles fueron los resultados?
Resultados en el conjunto de prueba 2022-2024, con 4 483 muestras y 219 excedencias:

| Modelo | PR-AUC | F1 | Recall | Precision |
|---|---|---|---|---|
| Persistencia | 0.594 | 0.555 | 0.553 | 0.558 |
| Regresión logística | 0.614 | 0.617 | 0.603 | 0.632 |
| MLP (5 semillas) | 0.658 ± 0.015 | 0.606 ± 0.035 | 0.544 ± 0.082 | 0.705 ± 0.078 |
| **CNN-1D (5 semillas)** | **0.673 ± 0.012** | **0.622 ± 0.013** | 0.553 ± 0.025 | **0.715 ± 0.049** |

- La CNN-1D supera a la persistencia con significancia estadística: ΔPR-AUC = +0.064, IC 95 % [0.012; 0.117], McNemar p = 0.0001.
- También supera a la regresión logística en PR-AUC: +0.044, IC 95 % [0.011; 0.078].
- Detecta lo mismo que la persistencia con 45 % menos falsas alarmas.
- No se diferencia significativamente del MLP, pero usa 4 veces menos parámetros.

## Estructura
```
data/raw/       CSV oficial (no se versiona; ver data/README.md)
notebooks/      notebook ejecutado
src/            datos_aire.py (preparación) y cnn1d_numpy.py (CNN en NumPy)
tests/          pruebas unitarias
prototipo/      alerta_pm10.py (línea de comandos)
modelos/        modelo exportado (.npz + .json)
resultados/     figuras, tablas y boletines
docs/           informe (Word) y referencias.bib
```

## Licencia
El código se publica con licencia MIT. Los datos pertenecen al SENAMHI y se usan bajo la licencia Open Data Commons Attribution (ODC-BY).
