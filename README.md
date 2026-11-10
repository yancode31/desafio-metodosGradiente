# desafio-metodosGradiente
================================================================================
UNIVERSIDAD MAYOR DE SAN ANDRÉS
FACULTAD DE CIENCIAS PURAS Y NATURALES · CARRERA DE INFORMÁTICA
MÉTODOS NUMÉRICOS · DESAFÍO ACADÉMICO INDIVIDUAL
================================================================================

Presupuesto de TI con Gradientes Conjugados Precondicionados

Nombre completo : YANDIRA ENILSE MORALES NINA
Carrera         : Informática
Mención         : Ingeniería de Sistemas
Materia         : Métodos Numéricos

================================================================================
1. DESCRIPCIÓN DEL CASO
================================================================================

Una empresa mediana distribuye su presupuesto anual de tecnología entre cuatro
unidades organizacionales. Cada unidad tiene un costo propio creciente
(rendimientos decrecientes) y comparte recursos con las demás (licencias,
personal, infraestructura).

Variables del modelo (en miles de Bs):

    x1  →  Desarrollo de software
    x2  →  Infraestructura y redes
    x3  →  Seguridad y soporte
    x4  →  Capacitación e I+D

Se busca el reparto x que minimiza

    f(x) = 1/2 · xᵀ A x − bᵀ x

Como A es simétrica definida positiva, f es estrictamente convexa y su único
mínimo satisface

    A x = b

Los valores utilizados son ilustrativos, calibrados para el ejercicio académico.
No corresponden a cifras oficiales de ninguna empresa.


================================================================================
2. FORMULACIÓN MATEMÁTICA
================================================================================

Sistema lineal:

    A · x = b ,   A ∈ ℝ^(4×4) ,   x , b ∈ ℝ^4

Interpretación de coeficientes:

    a_ii   →  costo marginal propio de la unidad i
    a_ij = a_ji  (i≠j)  →  competencia por recurso compartido entre i y j
    b_i    →  necesidad de inversión de la unidad i

Propiedades que debe cumplir A:

    - Simétrica:  A = Aᵀ
    - Definida positiva: todos los menores principales líderes > 0
      (criterio de Sylvester) o, equivalentemente, autovalores positivos

La aplicación web valida ambas condiciones y notifica al usuario si falla.


================================================================================
3. TRES ESCENARIOS OBLIGATORIOS
================================================================================

Escenario             Construcción                          Análisis esperado
--------------------------------------------------------------------------------
Ideal                 A = A0 , b = b0                        Convergencia rápida
                      κ(A) ~ 10¹                             y exactitud alta

Bajo estrés           A' = 1.15 · A0   (inflación)            κ(A) no cambia;
                      b' = demanda ajustada                  solo se desplaza
                                                             el equilibrio

Mal condicionado      A' = D · A · D ,  b' = D · b           κ(A) ~ 10⁹ por
                      D = diag(1, 10, 100, 1000)             mal escalamiento
                                                             diagonal; Jacobi
                                                             lo reduce mucho


================================================================================
4. MÉTODOS IMPLEMENTADOS
================================================================================

4.1 Solución de referencia — Cholesky

    A = L Lᵀ ,   L y = b ,   Lᵀ x = y

    Válido por ser A simétrica definida positiva.
    Se clasifica como solución exacta salvo redondeo de máquina.

4.2 Gradientes Conjugados (GC)

    α_k = (r_kᵀ r_k) / (p_kᵀ A p_k)
    β_k = (r_{k+1}ᵀ r_{k+1}) / (r_kᵀ r_k)

4.3 Gradientes Conjugados Precondicionados (GCP)
    Algoritmo de Suñagua (2020):

    M z_k = r_k
    α_k = (r_kᵀ z_k) / (p_kᵀ A p_k)
    β_k = (r_{k+1}ᵀ z_{k+1}) / (r_kᵀ z_k)

4.4 Precondicionadores disponibles

    Jacobi         M = diag(A)
                   Caso canónico de mal escalamiento diagonal.

    IC(0)          M = L̃ L̃ᵀ  (Cholesky incompleto sin relleno)
                   Útil con fuerte acoplamiento fuera de la diagonal.

    SSOR (ω=1)     M = (1/(2−ω)) · (D+ωL) · D⁻¹ · (D+ωL)ᵀ
                   Alternativa simétrica al IC(0).

    Ninguno        M = I
                   Solo para contraste / depuración.

4.5 Criterio de parada

    ‖r_k‖₂ / ‖b‖₂ < 10⁻⁶ ,   r_k = b − A x_k
    N_máx = 1000 iteraciones
    x0 = [0, 0, 0, 0]ᵀ  por defecto


================================================================================
5. MÓDULOS DE LA PÁGINA WEB
================================================================================

  #    Módulo                            Descripción
--------------------------------------------------------------------------------
  1    Inicio                            Título, estudiante, mención,
                                         síntesis del caso
  2    Caso de estudio                   Contexto financiero y justificación
  3    Formulación                      Variables, sistema Ax=b,
                                         interpretación de coeficientes
  4    Entrada de datos                  Edición de A y b, x0, tolerancia,
                                         precondicionador
  5    Tres escenarios                   Botones Ideal / Estrés / Mal cond.
  6    Solución de referencia            Cholesky + Sylvester + λmin
  7    GCP vs GC                         Vectores, iteraciones, residuos
  8    Análisis de errores               Ea, Er, Ep por variable
  9    Tabla comparativa obligatoria     12 filas del enunciado por escenario
 10    Módulo QA                         5 pruebas con formato estándar
 11    Conclusiones y referencias        Interpretación matemática y financiera

La página recalcula todo automáticamente al modificar cualquier coeficiente.


================================================================================
6. ANÁLISIS DE ERRORES Y CONDICIONAMIENTO
================================================================================

Por variable i:

    Error absoluto     Ea,i = |x_ref,i − x_GCP,i|
    Error relativo     Er,i = Ea,i / |x_ref,i|      (no definido si x_ref,i = 0)
    Error porcentual   Ep,i = Er,i × 100 %

Globales:

    Residuo relativo   ‖b − A x_GCP‖₂ / ‖b‖₂
    Condición          κ(A) = ‖A‖ · ‖A⁻¹‖
                       κ(M⁻¹A) para la matriz precondicionada


================================================================================
7. PRUEBAS DE SOFTWARE (QA)
================================================================================

Se ejecutan desde el módulo 9 con el botón "Ejecutar pruebas". Formato estándar:

  #     Prueba                    Entrada                  Esperado
--------------------------------------------------------------------------------
  1     Carga de escenarios       3 escenarios, Jacobi,    Todos convergen con
                                  tol = 1e-6               residuo < 1e-6

  2a    Matriz no simétrica       a12 = 2 (original 1)     Error capturado sin
                                                           colapsar

  2b    Matriz no def. positiva   a11 = −1                 Error capturado sin
                                                           colapsar

  3     Edición dinámica          a11: 4 → 6               Solución cambia y
                                                           residuo < 1e-6

  4     Comparativa precond.      Mal condicionado         GCP ≤ iteraciones
                                                           de GC


================================================================================
8. GUÍA RÁPIDA DE USO
================================================================================

    1. Ver un escenario:
       Clic en "Ideal", "Bajo estrés" o "Mal condicionado" (sección 4).

    2. Cambiar precondicionador:
       Selector en la sección 3; el cálculo se actualiza al instante.

    3. Editar A o b:
       Modificar cualquier celda; todo se recalcula automáticamente.

    4. Ajustar tolerancia o N máx:
       Modificar los campos correspondientes en la sección 3.

    5. Ejecutar pruebas QA:
       Botón "Ejecutar pruebas" en la sección 9.

    6. Verificar errores:
       Sección 7 y tabla comparativa de la sección 8.


================================================================================
9. RESULTADOS ESPERADOS
================================================================================

    Ideal
        κ(A) ~ 10¹.
        GC y GCP convergen en ≤ 4 iteraciones (n = 4).

    Bajo estrés
        κ(A) igual al Ideal.
        La solución se desplaza según la nueva demanda.

    Mal condicionado
        κ(A) ~ 10⁹.
        GC sin precondicionar es inestable o lento.
        GCP-Jacobi reduce κ(M⁻¹A) varios órdenes de magnitud y converge
        en pocas iteraciones.


================================================================================
10. LIMITACIONES
================================================================================

    - Tamaño de sistema fijo en n = 4 (suficiente para el desafío).
    - Aritmética de doble precisión (IEEE 754).
    - Los datos son sintéticos, no representan cifras contables reales.
    - El cálculo de κ(M⁻¹A) se realiza por simetrización de M⁻¹A; es una
      aproximación estable pero no idéntica a la definición formal con normas
      matriciales.
