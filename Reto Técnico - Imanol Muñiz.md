## Contexto

**Radii** es una startup que ofrece Supply Chain as a Service, usando IA y aprovechando el nearshoring en México.

**El problema que resolvemos**: Automatizamos todo el flujo desde que un cliente nos envía un email pidiendo una pieza, hasta la entrega física. Esto incluye cotizar, encontrar proveedores, gestionar producción y logística.

**Industrias**: Aerospace, Automotive, Industrial

**Tu reto es en el dominio de Quoting**: Usamos un modelo de IA para generar cotizaciones automáticas a partir de planos técnicos. El modelo está en producción, pero Finanzas detectó que los márgenes de las cotizaciones generadas por IA son **12% más bajos** que las cotizaciones humanas. Tu trabajo es encontrar por qué.

---

## El Reto

En tu entrevista hablamos del cotizador y tú propusiste usar datos históricos para entrenar modelos de regresión, e identificaste edge cases de feasibility. Este reto va por ese lado — pero en lugar de construir el modelo, vas a **auditar sus outputs**.

**Auditar un batch de 10 cotizaciones generadas por IA, encontrar los errores, explicar por qué los márgenes están bajos, y proponer reglas de validación para evitar que vuelva a pasar.**

---

## Los Datos

### Reglas de referencia

Estas son las reglas que el equipo de operaciones usa para cotizar. El modelo de IA debería seguirlas, pero no siempre lo hace.

```
DENSIDADES DE MATERIALES (g/cm³)
Aluminum 6061: 2.70    Aluminum 7075: 2.81
Steel 1018: 7.85       Steel 4140: 7.85
Titanium: 4.43
Stainless 304: 8.00    Stainless 316: 8.00

REGLA DE REDONDEO DE STOCK
Redondear HACIA ARRIBA cada dimensión al múltiplo de 10mm más cercano
antes de calcular peso y costo de material.
Ejemplo: 102 × 98 × 47 → 110 × 100 × 50

REGLAS DE NEGOCIO
1. Precio debe exceder (costo_material_stock × 1.15) + mano_de_obra
2. Peso cotizado debe reflejar dimensiones de STOCK (después del redondeo)
3. Titanio: surcharge de tooling de $150 USD, amortizado sobre el batch
4. Descuento por volumen: 15-20% off en cantidades de 10+
5. Premium por urgencia: +25% si el lead time es < 5 días
```

### Batch de cotizaciones a auditar

| ID | Cliente | Material | Forma | Dimensiones (mm) | Qty | Peso (kg) | Precio/kg Est. (USD) | Precio Total IA | Lead Time |
|:---|:---|:---|:---|:---|:---:|:---:|:---:|:---:|:---:|
| Q1 | Acme Corp | Aluminum 6061 | Caja | 102 × 98 × 47 | 1 | 1.26 | $7.50 | $185 | 10 días |
| Q2 | Acme Corp | Aluminum 6061 | Caja | 102 × 98 × 47 | 25 | 1.26 | $7.50 | $185 | 10 días |
| Q3 | BetaTech | Steel 4140 | Cilindro | Ø80 × 200 | 5 | 2.70 | $4.75 | $290 | 7 días |
| Q4 | BetaTech | Steel 1018 | Cilindro | Ø80 × 200 | 5 | 7.90 | $2.25 | $285 | 7 días |
| Q5 | CanDo Mfg | Titanium | Caja | 2.0 × 2.0 × 4.0 | 1 | 1.10 | $62.00 | $320 | 14 días |
| Q6 | CanDo Mfg | Titanium | Caja | 50 × 50 × 100 | 20 | 1.10 | $62.00 | $145 | 14 días |
| Q7 | DeltaAero | Aluminum 7075 | Caja | 203 × 152 × 22 | 50 | 2.10 | $15.00 | $95 | 3 días |
| Q8 | EastMech | Stainless 316 | Cilindro hueco | Ø60 (p5) × 150 | 10 | 2.00 | $9.50 | $175 | 8 días |
| Q9 | EastMech | Stainless 304 | Cilindro hueco | Ø60 (p5) × 150 | 10 | 2.00 | $6.50 | $210 | 8 días |
| Q10 | Acme Corp | Aluminum 6061 | Caja | 102 × 98 × 47 | 1 | 1.26 | $7.50 | $140 | 3 días |

### Contexto adicional

- El modelo fue entrenado con cotizaciones históricas ganadas
- En los meses 1-3, el modelo generó ~50 cotizaciones/mes con márgenes aceptables
- En los meses 4-6, el volumen subió a ~300 cotizaciones/mes y los márgenes cayeron
- La tasa de conversión de titanio es solo 8% (vs. 35% promedio general)

---

## Lo que debes entregar (en el notebook)

### Parte A — Detección de errores (lo más importante)

Para cada cotización:
- ¿El peso es correcto? Muestra tus cálculos.
- ¿El precio cumple las reglas de negocio?
- Clasifica cada quote: ✅ Correcto, ⚠️ Warning, 🛑 Crítico
- Estima el impacto en dinero donde sea posible

### Parte B — Análisis de patrones

1. ¿Por qué la tasa de conversión de titanio es solo 8%?
2. ¿Por qué los márgenes de la IA son 12% más bajos que los humanos?
3. ¿Qué pasó en los meses 4-6? ¿Qué le preguntarías al equipo?

### Parte C — Reglas de validación

Escribe **5 reglas** en Python que atrapen estos errores automáticamente.
Para cada regla incluye:
- El código
- Qué errores atrapa
- Estimación de tasa de falsos positivos
- Propón al menos una fuente de datos externa que mejoraría la validación

### Parte D — Preguntas sobre los datos de entrenamiento

Si pudieras hacerle **3 preguntas** al equipo de datos sobre cómo se entrenó el modelo, ¿cuáles serían? Explica qué esperas descubrir con cada una.

### Parte E — Reflexión de proceso

- ¿Qué herramientas usaste? (AI, calculadora, scripts, etc.)
- ¿Cuál fue la parte más difícil?
- Describe al menos un callejón sin salida que exploraste
- Confianza en tus respuestas (1-5) con justificación

---

## Requerimientos técnicos

### Obligatorio
- Jupyter Notebook (Google Colab o local)
- Cálculos explícitos (muestra tu trabajo, no solo conclusiones)
- Código de validación funcional en Python
- El notebook debe estar en un repo de GitHub

### Opcional (bonus)
- Visualizaciones que cuenten la historia
- Análisis adicional más allá de lo pedido
- Que tus reglas de validación manejen todas las geometrías (caja, cilindro, cilindro hueco)

### No necesitas
- Frontend ni UI
- Entrenar ningún modelo
- Deploy

---

## Entregables

1. **Repo en GitHub** con el notebook
2. **Transcripts de tus sesiones con AI** — Incluye los archivos de sesión de Claude Code (están en `~/.claude/projects/`) o exports de tus conversaciones con cualquier herramienta AI que hayas usado. Queremos leer exactamente cómo le diste contexto, cómo iteraste y cómo resolviste problemas.
3. **Video corto (2-3 min)** mostrando:
   - Tu proceso de auditoría (cómo abordaste el problema)
   - Los hallazgos más importantes
   - Cómo usaste AI en tu proceso
   - Súbelo como prefieras (Loom, YouTube, o adjunto por correo)

---

## Tiempo estimado

**4-6 horas** de trabajo. No esperamos algo perfecto, queremos ver cómo piensas y analizas.

Fecha límite: **lunes 16 de febrero**.

---

## Qué vamos a evaluar

| Criterio | Peso |
|----------|------|
| **Detección de errores (A)** - ¿Encontraste los errores y mostraste los cálculos? | 30% |
| **Análisis de patrones (B)** - ¿Entiendes por qué el modelo falla sistémicamente? | 20% |
| **Reglas de validación (C)** - ¿Tu código atrapa los errores? ¿Es robusto? | 15% |
| **Preguntas sobre datos (D)** - ¿Haces las preguntas correctas? | 10% |
| **Uso de AI** - ¿Cómo usaste herramientas AI en tu proceso? | 15% |
| **Pensamiento crítico (E)** - Reflexión honesta, calibración, limitaciones | 10% |

### Detalle: Uso de AI (15%)

Queremos ver cómo integras AI en tu flujo de análisis. Específicamente:

- **Exploración**: ¿Usaste AI para explorar los datos o verificar hipótesis?
- **Código**: ¿Cómo usaste AI para escribir las reglas de validación?
- **Verificación**: ¿Usaste AI para validar tus propios cálculos?
- **Herramientas avanzadas** (bonus): Claude Code, agents, MCPs, etc.

---

## Tips

- Este reto no es sobre programar un modelo — es sobre **auditar outputs de IA**. La habilidad que más importa en producción.
- Muestra tus cálculos. Una conclusión sin la cuenta que la respalda no cuenta.
- Si algo no cuadra, investiga por qué. No asumas que la IA tiene razón.
- Usa AI para ayudarte (Claude, ChatGPT, Copilot, lo que quieras). Queremos ver cómo lo usas, no que lo hagas sin ayuda.
- Si tienes dudas, pregunta. Preferimos que preguntes a que asumas algo incorrecto.

---

*Este reto es confidencial y solo para fines de evaluación.*
