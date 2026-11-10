# Presupuesto de TI con Gradientes Conjugados Precondicionado (GCP)

**Universidad Mayor de San Andrés · Facultad de Ciencias Puras y Naturales · Carrera de Informática**
Métodos Numéricos · Desafío académico individual

- **Estudiante:** Yandira Enilse Morales Nina
- **Mención:** Ingeniería de Sistemas


## 1. Qué resuelve
Una empresa mediana reparte su presupuesto anual de TI entre cuatro unidades (miles de Bs):
x₁ Desarrollo de software · x₂ Infraestructura y redes · x₃ Seguridad y soporte · x₄ Capacitación e I+D.

El reparto óptimo minimiza f(x) = ½·xᵀAx − bᵀx. Como A es simétrica definida positiva, f es estrictamente convexa y su único mínimo cumple **Ax = b**. Los datos son ilustrativos, no cifras oficiales.

## 2. Instalación y ejecución local
No requiere instalación ni dependencias.
1. Clona o descarga el repositorio.
2. Abre `index.html` en un navegador moderno (Chrome, Firefox, Edge).
3. Para publicarlo: Settings → Pages → rama `main`, carpeta raíz.

## 3. Arquitectura
Un solo archivo `index.html` (HTML + CSS + JavaScript, sin librerías externas), organizado en bloques:

| Bloque | Función |
|---|---|
| Datos y escenarios | A, b y los tres escenarios |
| Álgebra básica | producto punto, norma, determinante |
| `chol`, `fwd`, `bwd` | solución de referencia (Cholesky) |
| `eig` | autovalores (Jacobi cíclico) para κ(A) y κ(M⁻¹A) |
| `precond` | Jacobi, IC(0), SSOR (ω = 1) |
| `pcg` | GCP (Suñagua, 2020); con M = I es el GC simple |
| `validar` | simetría y criterio de Sylvester |
| `resolver` | une todo y calcula errores y residuo |
| Interfaz | los 10 módulos, gráfico de convergencia, QA |

## 4. Los tres escenarios
| Escenario | Construcción | κ(A) | Qué se observa |
|---|---|---|---|
| Ideal | A = A₀, b = b₀ | ≈ 2,3 | GC y GCP convergen en 4 iteraciones |
| Bajo estrés | A = 1,15·A₀ (inflación), b ajustado | ≈ 2,3 | κ no cambia; solo se desplaza el equilibrio |
| Mal condicionado | A = D·A₀·D, b = D·b₀, D = diag(1, 10, 100, 1000) | ≈ 9,3·10⁵ | Con Jacobi κ(M⁻¹A) ≈ 2,05; GCP 4 it. vs GC 5 it. |

## 5. Métodos
- **Referencia:** A = LLᵀ, Ly = b, Lᵀx = y (solución exacta salvo redondeo).
- **GC:** αₖ = rₖᵀrₖ / pₖᵀApₖ, βₖ = rₖ₊₁ᵀrₖ₊₁ / rₖᵀrₖ.
- **GCP (Suñagua, 2020):** Mzₖ = rₖ, αₖ = rₖᵀzₖ / pₖᵀApₖ, βₖ = rₖ₊₁ᵀzₖ₊₁ / rₖᵀzₖ.
- **Precondicionadores:** Jacobi (M = diag A), IC(0) (Cholesky incompleto sin relleno), SSOR con M = 1/(ω(2−ω))·(D+ωL)·D⁻¹·(D+ωL)ᵀ y ω = 1, o ninguno (M = I).
- **Parada:** ‖rₖ‖₂/‖b‖₂ < 10⁻⁶, Nmáx = 1000, x₀ = 0 por defecto.
- **Errores:** Ea, Er, Ep por variable (Er no definido si x_ref = 0), residuo relativo global, κ(A) y κ(M⁻¹A).

## 6. Módulos de la web
1 Caso · 2 Formulación · 3 Entrada de datos · 4 Escenarios · 5 Referencia · 6 GCP vs GC · 7 Errores (incluye reglas de negocio y flujo de caja interdepartamental) · 8 Tabla comparativa · 9 QA · 10 Conclusiones y referencias.

## 7. Pruebas (módulo 9, botón "Ejecutar pruebas")
| Prueba | Entrada | Esperado |
|---|---|---|
| 1 Carga de escenarios | 3 escenarios, Jacobi, tol 10⁻⁶ | Todos convergen, residuo < 10⁻⁶ |
| 2a Matriz no simétrica | a₁₂ = 2 (original 1) | Error capturado sin colapsar |
| 2b No definida positiva | a₁₁ = −1 | Error capturado sin colapsar |
| 3 Edición dinámica | a₁₁: 4 → 6 | La solución cambia y residuo < 10⁻⁶ |
| 4 Precondicionamiento | Mal condicionado | GCP ≤ iteraciones de GC |

## 8. Guía rápida
1. Elige un escenario con los botones de la sección 4.
2. Cambia el precondicionador, la tolerancia o Nmáx en la sección 3.
3. Edita cualquier celda de A o b: todo se recalcula al instante.
4. Revisa errores (sección 7) y la tabla comparativa (sección 8).

## 9. Limitaciones
- Tamaño fijo n = 4; doble precisión (IEEE 754).
- Datos sintéticos, no contables.
- Con n = 4, GC termina en pocas iteraciones por teoría; la ventaja de GCP es modesta y se nota más en κ(M⁻¹A) que en el conteo de iteraciones.

## 10. Referencias
- Suñagua S., P. (2020). Método de Gradientes Conjugados Precondicionado. *Revista Boliviana de Matemática – UMSA*, 04, 2–7.
- Golub, G. H., & Van Loan, C. F. (2013). *Matrix Computations* (4.ª ed.). Johns Hopkins University Press.
- Saad, Y. (2003). *Iterative Methods for Sparse Linear Systems* (2.ª ed.). SIAM.
