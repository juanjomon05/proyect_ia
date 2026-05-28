# Clasificador de Carros por Precio
### Proyecto de Inteligencia Artificial

**Integrantes:** Juan Jose Montoya · Isabel Acevedo · Miguel Aguilar
**Fecha:** Mayo 2026  
**Repositorio:** https://github.com/juanjomon05/proyect_ia

---

## De que trata este proyecto

La idea es simple: mostrarle una foto de un carro a una IA y que ella diga si ese carro es economico, de gama media o de lujo, sin ver el precio, solo mirando la imagen.

Entrenamos una red neuronal para que aprenda a reconocer las caracteristicas visuales que hacen que un carro parezca caro o barato: la forma de la carroceria, los faros, los rines, el diseno general. El modelo clasifica cada foto en una de estas tres categorias:

| Categoria | Precio estimado | Ejemplos |
|-----------|----------------|---------|
| Economico | $8,000 - $25,000 USD | Toyota, Honda, Hyundai, Chevrolet |
| Gama Media | $25,000 - $65,000 USD | BMW, Audi, Mercedes-Benz, Cadillac |
| Lujo | $65,000+ USD | Ferrari, Lamborghini, Bentley, Rolls-Royce |

---

## El Dataset

Usamos el **Stanford Cars Dataset**, un conjunto de datos con fotos de 196 modelos de autos distintos. Lo descargamos desde Kaggle y lo reorganizamos manualmente en las 3 categorias de precio segun la marca y el modelo de cada vehiculo.

### Cuantas fotos usamos

| | Economico | Gama Media | Lujo | Total |
|-|-----------|------------|------|-------|
| Entrenamiento | 4,111 | 2,748 | 1,285 | 8,144 |
| Prueba | 4,063 | 2,710 | 1,268 | 8,041 |

Un problema que encontramos desde el principio fue el desbalance: hay 3 veces mas fotos de carros economicos que de lujo. Esto hacia que el modelo aprendiera a predecir "economico" casi siempre. Mas adelante explicamos como lo resolvimos.

---

## Como funciona la IA por dentro

### Transfer Learning

Entrenar una red neuronal desde cero requiere millones de imagenes y dias de computo. En cambio, usamos una tecnica llamada **Transfer Learning**: tomamos un modelo que ya sabe reconocer imagenes (entrenado con 1.2 millones de fotos) y lo adaptamos a nuestra tarea especifica.

El modelo base se llama **EfficientNet-B0**. Es compacto, rapido y preciso para su tamanio. La idea fue congelar la mayoria de sus capas (que ya saben reconocer formas, texturas y bordes) y solo ajustar las ultimas dos junto con una capa de clasificacion nueva.

Solo entrenamos el 28% de los parametros del modelo, lo que hace el entrenamiento mucho mas rapido sin sacrificar precision.

### Preparacion de las imagenes

Antes de entrar al modelo, cada imagen se redimensiona a 224x224 pixeles y se normaliza. Durante el entrenamiento tambien se aplican variaciones aleatorias como volteo horizontal, rotacion leve y cambios de brillo, para que el modelo no memorice las fotos sino que aprenda a generalizar.

### El problema del desbalance

Como habia muchas mas fotos de carros economicos, el modelo tendia a predecir esa categoria casi siempre. Probamos varias aproximaciones hasta encontrar la combinacion que funcionaba:

| Intento | Resultado |
|---------|-----------|
| Entrenamiento normal | Predecia "economico" casi siempre |
| Sobre-muestreo de clases minoritarias | Se paso al extremo contrario: todo "lujo" |
| Pesos en la funcion de perdida | Funciono bien |
| Correccion de probabilidad en inferencia | Ajuste fino sin reentrenar |

La solucion final combina pesos de clase en el entrenamiento (el error en "lujo" pesa 3.2 veces mas que en "economico"), label smoothing para evitar que el modelo sea demasiado confiado, y una correccion de sesgo al momento de predecir.

---

## Resultados

El modelo entreno durante 5 epocas, mejorando consistentemente en cada una:

| Epoca | Precision en validacion |
|-------|------------------------|
| 1 | 53.8% |
| 2 | 58.2% |
| 3 | 61.2% |
| 4 | 63.3% |
| 5 | 64.1% |

### Desempeno por categoria

| Categoria | Precision | Recall | F1 |
|-----------|-----------|--------|----|
| Economico | 69% | 84% | 0.76 |
| Gama Media | 53% | 50% | 0.51 |
| Lujo | 71% | 31% | 0.43 |

La categoria economico es la que mejor resulta, lo cual tiene sentido dado que tiene mas ejemplos de entrenamiento. Gama media es la mas dificil de clasificar porque comparte caracteristicas visuales con las otras dos categorias. Lujo tiene buena precision pero bajo recall: cuando el modelo dice que un carro es de lujo suele tener razon, pero deja de detectar muchos carros de esa categoria.

El mayor reto fue distinguir entre gama media y lujo. Un BMW serie 7 y un Rolls-Royce pueden parecerse visualmente mas de lo que uno esperaria.

---

## Grad-CAM — en que se fijo el modelo

Una parte importante del proyecto fue poder entender que estaba mirando la IA al tomar cada decision. Para eso implementamos **Grad-CAM**, una tecnica que genera un mapa de calor sobre la imagen mostrando que zonas influyeron mas en la prediccion.

Las zonas rojas y naranjas son las que el modelo considero importantes; las azules y verdes, las que practicamente ignoro.

En la practica el modelo tiende a enfocarse en la forma de la carroceria, los faros y los rines, que son exactamente los elementos que visualmente distinguen un carro economico de uno de lujo. Eso confirma que la red aprendio caracteristicas relevantes y no esta detectando cosas al azar.

---

## La Interfaz

Para que cualquier persona pueda usar el modelo sin saber de programacion, construimos una interfaz web con **Gradio**. El usuario sube una foto, presiona el boton de clasificar y el sistema muestra la categoria estimada, el rango de precio, las probabilidades por clase y el mapa de calor Grad-CAM.

El diseno visual de la interfaz y la integracion de Grad-CAM fueron desarrollados con asistencia de **Claude (Anthropic)**.

Para usarla basta con ejecutar la ultima celda del notebook y abrir `http://127.0.0.1:7860` en el navegador. Si solo se quiere probar la interfaz sin reentrenar, el modelo ya esta guardado en el repositorio (`car_classifier.pth`).

---

## Tiempos de ejecucion

| Equipo | Tiempo de entrenamiento |
|--------|------------------------|
| CPU local | 20-25 minutos |
| GPU en la nube (Lightning AI / Colab) | 5-7 minutos |

---

## Conclusiones

Logramos un sistema funcional que clasifica fotos de carros con un 64% de precision en 3 rangos de precio, con una interfaz visual intuitiva y mapas de atencion que explican cada decision.

El Transfer Learning demostro ser muy efectivo: con un dataset moderado y pocas horas de entrenamiento se obtienen resultados utiles. El desbalance de clases fue el principal obstaculo y requirio varias iteraciones para encontrar la combinacion correcta de tecnicas.

Como trabajo futuro, lo mas util seria ampliar el dataset de carros de lujo y gama media para equilibrar mejor las clases, y posiblemente probar modelos mas grandes con acceso a GPU.

---

## Tecnologias usadas

| Herramienta | Para que se uso |
|-------------|----------------|
| Python 3.13 | Lenguaje base |
| PyTorch | Entrenamiento de la red neuronal |
| EfficientNet-B0 (torchvision) | Modelo base preentrenado |
| pytorch-grad-cam | Mapas de calor Grad-CAM |
| Gradio | Interfaz web interactiva |
| scikit-learn | Metricas de evaluacion |
| kagglehub | Descarga del dataset |

---

## Referencias

- Tan & Le (2019). *EfficientNet: Rethinking Model Scaling for CNNs*. ICML.
- Selvaraju et al. (2017). *Grad-CAM: Visual Explanations from Deep Networks*. ICCV.
- Dataset: Stanford Cars — https://www.kaggle.com/datasets/jutrera/stanford-car-dataset-by-classes-folder
