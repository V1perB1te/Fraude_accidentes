# Guía Paso a Paso: Análisis y Modelado de Detección de Fraude

Este documento es el mapa de ruta definitivo para el proyecto, diseñado para superar las bajas correlaciones lineales y maximizar la detección de fraude mediante modelos avanzados.

---

### Paso 1: Refinamiento de Datos (Finalizar Limpieza)
*   **Objetivo**: Asegurar que el dataset esté 100% listo y sea numérico.
*   **Acciones**:
    *   Convertir las variables de texto restantes (`Make`, `PolicyType`, `VehicleCategory`, `BasePolicy`).
    *   Confirmar que `Age = 0` ha sido reemplazado correctamente por la mediana.
    *   Eliminar definitivamente columnas redundantes (ej: `AgeOfPolicyHolder`).

### Paso 2: Ingeniería de Características (Búsqueda de Señales)
*   **Objetivo**: Crear "super-variables" combinando columnas para encontrar correlaciones más fuertes.
*   **Acciones**:
    *   Crear interacciones lógicas: `Culpa_Costo` (Fault + VehiclePrice), `Joven_Deportivo` (Age + VehicleCategory), etc.
    *   Calcular la correlación de estas nuevas variables con `FraudFound_P`.
    *   Visualizar patrones mediante agrupaciones (Groupby) para identificar "hotspots" de fraude.

### Paso 3: Preparación para Modelado (Preprocessing)
*   **Objetivo**: Transformar los datos para que los algoritmos los procesen eficientemente.
*   **Acciones**:
    *   Aplicar **One-Hot Encoding** a las variables nominales seleccionadas.
    *   Realizar el **Escalado de características** (StandardScaler) para normalizar rangos.

### Paso 4: División Estratégica de Datos
*   **Objetivo**: Validar el modelo en un entorno realista.
*   **Acciones**:
    *   Dividir en Entrenamiento (80%) y Testeo (20%).
    *   **Importante**: Usar `stratify` para mantener la proporción de fraudes (~6%) en ambos sets.

### Paso 5: Balanceo de Clase (SMOTE)
*   **Objetivo**: Enseñar al modelo a reconocer el fraude (clase minoritaria).
*   **Acciones**:
    *   Aplicar **SMOTE** solo al conjunto de **Entrenamiento**.
    *   Mantener el conjunto de **Testeo desbalanceado** (sin SMOTE) para reflejar la realidad.

### Paso 6: Implementación de Modelos No Lineales
*   **Objetivo**: Usar algoritmos potentes que detecten patrones complejos.
*   **Acciones**:
    *   **Prioridad 1**: Random Forest y XGBoost (especialistas en datos tabulares).
    *   **Prioridad 2**: Regresión Logística (para tener un punto de comparación).

### Paso 7: Evaluación y Visualización de Resultados
*   **Objetivo**: Medir qué tan bien detectamos el fraude.
*   **Acciones**:
    *   Generar Matrices de Confusión.
    *   Gráficas de Curva ROC y Precision-Recall (clave para datos desbalanceados).
    *   Análisis de **Recall**: ¿Cuántos fraudes reales se nos escaparon?

### Paso 8: Análisis Final y Conclusiones
*   **Objetivo**: Extraer valor del análisis.
*   **Acciones**:
    *   Identificar las variables más importantes (Feature Importance).
    *   Documentar las conclusiones sobre el perfil del defraudador.
