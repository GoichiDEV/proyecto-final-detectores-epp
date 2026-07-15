# Proyecto Final: ¿Qué detector desplegar y por qué?

## Integrante

- **Bastian Marcelo Aguilera Villagra**


## Objetivo

Comparar tres paradigmas para detectar frames con al menos una persona sin casco:

1. YOLOv8n cerrado y afinado.
2. YOLO-World de vocabulario abierto.
3. VLM: Florence-2 para cajas y Moondream2 para la pregunta-evento.

Los tres sistemas se evaluaron sobre los mismos **34 frames** del test de CSS-Data, con semilla `2026`.

## Resultados

| Sistema | mAP@0.5 Person/Hardhat | IoU medio TP | Precisión evento | Recall evento | F1 evento | Latencia | FPS |
|---|---:|---:|---:|---:|---:|---:|---:|
| YOLOv8n cerrado | 0.360 | 0.759 | 0.800 | 0.800 | 0.800 | 61.0 ms | 16.40 |
| YOLO-World | 0.197 | 0.784 | 0.357 | 1.000 | 0.526 | 23.9 ms | 41.79 |
| Florence-2 + Moondream2 | 0.225* | 0.763 | 0.238 | 1.000 | 0.385 | 5144.8 ms | 0.19 |

\* Florence-2 no entrega una confianza calibrada en esta salida. La mAP usa scores unitarios y debe interpretarse junto con IoU y las métricas de evento.

## Robustez

### Sensibilidad al prompt

Se probaron cinco redacciones equivalentes:

- F1 medio: **0.298**
- Desviación estándar F1: **0.075**
- Recall medio: **0.600**
- Desviación estándar recall: **0.283**
- Acuerdo medio por frame: **0.665**
- Acuerdo total en frames: **2.9%**

### Clase o escena no vista

El YOLO cerrado no puede emitir la clase `tie` porque no forma parte de su cabeza de salida. YOLO-World puede añadirla por texto sin reentrenamiento.

## ¿Qué sistema desplegaríamos?

Se recomienda **YOLO-World**. Obtuvo recall de evento 100.0%, F1 0.526 y 41.79 FPS. La regla prioriza el recall porque un falso negativo deja pasar una infracción real y, entre alternativas con recall cercano al máximo, selecciona la más rápida. Además, supera el objetivo de 30 FPS en esta T4.

## Análisis crítico

### Negación

YOLO cerrado resuelve “sin casco” porque el dataset incluye `NO-Hardhat`. YOLO-World y Florence detectan objetos positivos y requieren una relación espacial. Moondream responde directamente, pero cambia con la redacción.

### Cámara de 30 FPS

Los modelos que no alcancen 30 FPS deben muestrear frames, procesar en paralelo o usar más hardware. El VLM es más adecuado para auditoría muestreada que para revisar cada frame en una sola T4.

### Nueva clase: guantes

YOLO cerrado requiere imágenes etiquetadas y reentrenamiento. YOLO-World y los VLM pueden probar el concepto por texto, aunque deben validarse antes de confiar en ellos.

### Licencias indicadas en la pauta

- Ultralytics YOLO: AGPL-3.0.
- YOLO-World: GPL-3.0.
- Florence-2: MIT.
- Moondream2: Apache-2.0.
- CSS-Data: CC BY 4.0.

Las condiciones actuales deben revisarse antes de un uso comercial.

### Costo

El análisis de costo es cualitativo, porque no se dispone de una estimación monetaria homogénea para los tres sistemas. Se comparan costos relativos: entrenamiento y etiquetado para YOLO cerrado; adaptación rápida pero mayor incertidumbre en vocabulario abierto; mayor costo de inferencia y menor FPS para el VLM.

## Limitaciones

- La muestra balanceada facilita la comparación, pero no representa la prevalencia real.
- Las relaciones persona-casco son heurísticas.
- Florence no entrega scores calibrados.
- Moondream puede responder de forma ambigua.
- CUDA no garantiza determinismo perfecto.
- Colab puede desconectarse.

- ## Videos

- Demostración de los tres paradigmas: https://youtu.be/6BvoNEZ_rc4
- Justificación de la decisión de despliegue: https://youtu.be/nxyWD_41GCc
- Demostración adicional: https://youtube.com/shorts/ifc5NYl4dh8?feature=share

## Archivos

- `Proyecto_Final_Detectores_EPP.ipynb`
- `results/comparison_summary.csv`
- `results/moondream_prompt_metrics.csv`
- `results/demo_tres_sistemas_h264.mp4`
