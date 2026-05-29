# Clasificador de Carros por Precio

Sistema de clasificación de imágenes de autos usando **Transfer Learning con EfficientNet-B0**. Dado una foto de un carro, el modelo estima su rango de precio y muestra qué zonas de la imagen influyeron en la decisión (Grad-CAM).

## Integrantes

-Juan Jose Montoya
-Miguel Angel Aguilar
-Isabel Acevedo
## Categorías

| Categoría | Rango de precio | Ejemplos |
|-----------|----------------|---------|
| Económico | $8,000 – $25,000 USD | Toyota, Honda, Hyundai, Chevrolet |
| Gama Media | $25,000 – $65,000 USD | BMW, Audi, Mercedes-Benz, Cadillac |
| Lujo | $65,000+ USD | Ferrari, Lamborghini, Bentley, Rolls-Royce |

## Demo

La interfaz permite subir cualquier foto de un carro y obtiene:
- Categoría de precio estimada
- Porcentaje de confianza por clase
- Mapa de calor Grad-CAM (zonas rojas = más peso en la decisión)

## Tecnologías

- **PyTorch** + **torchvision** — entrenamiento y modelo
- **EfficientNet-B0** — backbone preentrenado en ImageNet
- **pytorch-grad-cam** — interpretabilidad visual
- **Gradio** — interfaz web interactiva
- **Stanford Cars Dataset** (Kaggle) — 8,144 imágenes de entrenamiento

## Resultados

| Métrica | Valor |
|---------|-------|
| Accuracy | 64.15% |
| Macro F1 | 0.568 |
| Recall Económico | 84% |
| Recall Gama Media | 50% |
| Recall Lujo | 31% |

## Instalación

```bash
pip install torch torchvision torchaudio
pip install scikit-learn matplotlib seaborn pandas pillow
pip install grad-cam
pip install kagglehub
pip install gradio
```

## Uso

El modelo ya está entrenado e incluido (`car_classifier.pth`). Solo se puede ejecutar la ultima celda y se ejecutara la ventana del programa

```
http://127.0.0.1:7860
```

Sube una foto, clic en **Clasificar carro** y el resultado aparece con el mapa de atención.

## Estructura del proyecto

```
proyect_ia/
├── car_price_classification_notebook.ipynb   # Notebook principal
├── car_classifier.pth                        # Pesos del modelo entrenado
├── dataset/
│   ├── train/
│   │   ├── economico/
│   │   ├── gama_media/
│   │   └── lujo/
│   └── test/
│       ├── economico/
│       ├── gama_media/
│       └── lujo/
└── README.md
```
## Uso de Inteligencia Artificial

La interfaz visual de este proyecto fue desarrollada con asistencia de IA:

- **Diseño de la interfaz Gradio** (colores, layout, tarjetas de resultado, barras de confianza) — generado con apoyo de Claude (Anthropic)
- **Integración de Grad-CAM** en la interfaz — implementado con apoyo de Claude (Anthropic)

## Dataset

**Stanford Cars Dataset** 

196 modelos de autos reorganizados en 3 categorías:

| Split | Económico | Gama Media | Lujo | Total |
|-------|-----------|------------|------|-------|
| Train | 4,111 | 2,748 | 1,285 | 8,144 |
| Test | 4,063 | 2,710 | 1,268 | 8,041 |

## Video Explicativo 

https://www.youtube.com/watch?v=p-z-aTSjG-k
