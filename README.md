# Forterra

Sistema de detección de fraude en transacciones con tarjeta: dada una transacción, devolverá una probabilidad de fraude, una decisión (aprobar / revisar / bloquear) y una explicación legible. Proyecto de aprendizaje construido por iteraciones.

**Estado actual (M1):** baseline y regresión logística (pandas + scikit-learn), evaluados con split temporal frente a split aleatorio.

## Resultados M1

| Modelo | AP temporal | AP aleatorio | ROC-AUC temporal | ROC-AUC aleatorio |
|---|---|---|---|---|
| Baseline (`amt`) | 0.1729 | 0.1793 | 0.8454 | 0.8241 |
| Regresión logística | 0.1736 | 0.1750 | 0.8526 | 0.8361 |

AP = average precision. El AP de un modelo sin habilidad es la tasa de fraude: 0.0049 (temporal) y 0.0058 (aleatorio). Resultados con una sola semilla aleatoria (`random_state=42`).

## ¿Por qué la accuracy no sirve?

Tomemos dos modelos para un mismo dataset con un 99,5 % de transacciones genuinas. El modelo A marca todas las transacciones como genuinas: su accuracy es del 99,5 %, un rendimiento excelente con estándares normales, pero no detecta ni un solo fraude, así que rinde igual que si no hubiera modelo. El modelo B detecta el 100 % de los fraudes pero marca como fraudulentas el 5 % de las transacciones; tiene bastantes falsos positivos y una accuracy de ≈95,5 %, por debajo de la de A. Según la accuracy, A es mejor que B, cuando solo B sirve para encontrar fraudes.

## ¿Por qué el split aleatorio puede inflar los resultados?

En un modelo más completo, un split aleatorio puede inflar las métricas: al mezclar los datos, el modelo se evalúa en validación con transacciones de tarjetas de las que ya ha visto otras transacciones en entrenamiento. Es como estudiar con el ejercicio que luego cae en el examen: memoriza en lugar de generalizar.

En este primer modelo no se observa ese efecto (ROC-AUC 0.8526 temporal frente a 0.8361 aleatorio; AP 0.1736 frente a 0.1750). No puede explotarlo, porque no se usó `cc_num` como variable y el modelo no tiene forma de identificar una tarjeta. Pero sí aprendió algo: un ROC-AUC de ≈0.84–0.85 demuestra que ordena razonablemente bien las transacciones fraudulentas frente a las genuinas. El problema es que `amt` es su peso dominante, así que marca como fraude los importes más elevados, mientras que los fraudes reales están en importes medios-altos. Por eso, con umbral 0.5, no detecta ningún fraude en el split temporal.

## Cómo ejecutarlo

```bash
git clone https://github.com/astro717/Forterra.git
cd Forterra
python -m venv forterraenv
source forterraenv/bin/activate
pip install -r requirements.txt
```

Descarga `fraudTrain.csv`, colócalo en `data/` (no se sube al repo) y abre `data/make_dataset.ipynb` con `jupyter notebook`.
