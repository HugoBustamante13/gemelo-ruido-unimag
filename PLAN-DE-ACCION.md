# Plan de acción — Gemelo digital de ruido con visión 3D

Universidad del Magdalena · documento de trabajo

## 1. Dónde estamos

Ya existe, en esta carpeta:

| Archivo | Qué es | Estado |
|---|---|---|
| `index.html` | Panel 2D: mapa real/digital, nivel en vivo, 24 h, patrón semanal, estado del nodo, eventos | Funciona con datos simulados |
| `index3d.html` | Lo mismo con mapa inclinado y edificios en volumen | Funciona con datos simulados |
| `mar-caribe-3d.html` | Modelo 3D del edificio Mar Caribe | Medidas estimadas desde fotos |
| `cafeteria-3d.html` | Modelo 3D de la cafetería principal | Medidas estimadas desde fotos |

Lo que **todavía no está probado ni hecho**:

- Ninguna página se ha conectado a un broker MQTT real ni al endpoint `/api/historico`.
- El centro del mapa y el polígono de la cafetería son aproximados; no hay más edificios trazados.
- Los modelos 3D no están colocados sobre el mapa: son páginas aparte.
- Las medidas de los dos edificios son a ojo.
- No se ha probado en celular.

## 2. Plan por fases

El orden importa: primero que los datos sean reales y creíbles, después que se vea bien. Las duraciones son estimaciones para una o dos personas a tiempo parcial.

### Fase 1 — Datos reales de punta a punta (≈ 1 semana)

Es la fase que convierte la maqueta en un sistema.

1. Activar en Mosquitto un listener WebSocket (puerto 9001) de solo lectura para `campus/ruido/#`.
2. Confirmar que el nodo publica el JSON acordado: `zona, lat, lon, leq, lmax, seq, ts`. Decidir si `ts` va en segundos o milisegundos (el panel acepta ambos).
3. Construir el endpoint `/api/historico` en la Raspberry Pi, que consulte la base de datos y devuelva `agg1m` y `agg15m`. Las credenciales se quedan en el servidor.
4. Servir las páginas desde la Pi y abrirlas desde otro equipo de la red: debe desaparecer "DATOS SIMULADOS".
5. Probar los casos de falla: apagar el nodo, cortar la red, reenviar datos desde la SD. Verificar que el panel muestre pérdidas, recuperaciones y "sin datos".

**Listo cuando:** el panel lleva 48 h mostrando datos reales de la cafetería, con histórico y sin intervención manual.

### Fase 2 — Que la medida sea creíble (≈ 1–2 semanas, en paralelo)

Sin esto, cualquier evaluador técnico puede desmontar la propuesta.

1. Documentar la cadena de medida: micrófono, frecuencia de muestreo, ponderación A, tiempo de integración del Leq.
2. Calibrar contra un sonómetro de referencia (clase 1 o 2; preguntar en los laboratorios de la universidad). Medir lado a lado en varios niveles y reportar el error en dB.
3. Aplicar la corrección obtenida en el firmware o en el servidor, y guardar la fecha de calibración.
4. Decidir la ubicación del nodo: altura, distancia a paredes, protección contra lluvia y viento. La Resolución 627 de 2006 fija condiciones de medición (Anexo 3); revisarlas y decir explícitamente en cuáles se cumple y en cuáles no.
5. Redactar la advertencia honesta: el nodo es de monitoreo **indicativo y continuo**, no reemplaza una medición oficial con sonómetro certificado.

**Listo cuando:** hay una tabla "nodo vs. sonómetro" con el error medido y una ficha de instalación del nodo.

### Fase 3 — Campus georreferenciado (≈ 1 semana)

1. Corregir el centro del mapa en `CONFIG.mapa.centro`.
2. Trazar los edificios principales con el botón "Trazar edificio" (en vista 2D) y pegarlos en `EDIFICIOS`. Empezar por 8–12: Mar Caribe, cafetería, Bienestar, bloque Río Magdalena, biblioteca, los bloques de techo blanco y los de techo rojo.
3. Asignar a cada uno su `altura` (pisos × 3,5 m es suficiente al principio).
4. Alternativa más rápida: exportar las huellas desde OpenStreetMap y corregir solo las que falten.
5. Poner las coordenadas reales de cada nodo.

**Listo cuando:** al inclinar el mapa se reconoce el campus sin leer los rótulos.

### Fase 4 — Visión 3D integrada (≈ 1–2 semanas)

1. Medir en sitio o con planos los dos edificios ya modelados y corregir `CONFIG.edificio` en cada visor. Confirmar las dudas pendientes: de qué lado va el cilindro del Mar Caribe y la disposición de cocina, arcos y terraza en la cafetería.
2. Enlazar mapa y modelos: clic en un edificio del mapa abre su visor 3D.
3. Opcional: insertar los modelos detallados dentro del mapa 3D (capa personalizada de MapLibre con Three.js), ya con coordenadas y orientación reales.
4. Modelar uno o dos edificios más donde haya nodos.
5. Probar en celular y en un portátil modesto; bajar detalle si va lento.

**Listo cuando:** desde el mapa se llega a cada edificio con sensor y se ve su nivel sobre el modelo.

### Fase 5 — Más nodos y mapa de ruido (según presupuesto)

1. Pasar de 1 a mínimo 3–4 nodos: con menos de 3 no hay mapa de calor.
2. Elegir ubicaciones que contrasten: cafetería (ruidosa), biblioteca (silenciosa), un bloque de aulas, un borde del campus hacia la vía.
3. Advertir en el panel que el mapa de calor entre nodos es una interpolación visual, no una medición.

### Fase 6 — Vuelo de dron (opcional, si se consigue)

1. Permisos: universidad y normativa de la Aerocivil.
2. Vuelo en cuadrícula con traslape del 75–80 %, cenital más oblicuo.
3. Procesar en WebODM: ortofoto y modelo de elevación.
4. Usar la ortofoto como fondo del mapa y sacar las alturas reales de los edificios.

### Fase 7 — Despliegue para la presentación (≈ 3 días)

1. Descargar las librerías a la Pi en lugar de usar CDN, para no depender de internet en la sala (las imágenes satelitales sí lo necesitan; tener un plan B).
2. Dejar el servicio arrancando solo al encender la Pi.
3. Grabar un video de respaldo de 2 minutos con el sistema funcionando.
4. Ensayar la demostración completa al menos dos veces en el lugar donde se va a presentar.

## 3. Qué más hace falta para una buena propuesta

Lo técnico es la mitad. Esto es lo que suele decidir una evaluación:

### 3.1 Justificar por qué es un "gemelo digital" y no un tablero

Es la objeción más probable. Un tablero muestra datos; un gemelo además **modela y anticipa**. Agregar al menos una de estas capacidades:

- **Predicción:** estimar el nivel de las próximas horas a partir del patrón semanal y mostrarlo como línea punteada en la gráfica de 24 h.
- **Escenarios:** "¿qué pasa si se mueve el horario de almuerzo?" o "¿si se pone una barrera acústica?", aunque sea con un modelo simple de atenuación por distancia.
- **Alertas con acción:** aviso a Bienestar o a la dependencia responsable cuando un evento supera 5 minutos, y registro de qué se hizo.

### 3.2 Mostrar el problema con datos propios

- Una o dos semanas de mediciones reales de la cafetería: cuántas horas se supera el límite, a qué horas, qué días.
- Comparar con la referencia normativa (65 dB(A) diurno y 55 nocturno; confirmar el sector de la Resolución 627 que aplica a un campus) y con recomendaciones de la OMS para espacios de estudio.
- Una frase de impacto con número: "la cafetería supera el límite X horas por semana".

### 3.3 Validación

- La tabla de calibración de la fase 2.
- Disponibilidad del sistema: porcentaje de paquetes recibidos y recuperados desde la SD en una semana.
- Si es posible, una encuesta corta a estudiantes sobre percepción del ruido en las mismas zonas, para cruzarla con las mediciones.

### 3.4 Privacidad y ética

- Dejar explícito que el nodo **no graba ni transmite audio**, solo niveles en decibelios.
- Poner un aviso visible junto a cada nodo.
- Conseguir el aval de la dependencia que corresponda (Bienestar, Planeación o Seguridad y Salud en el Trabajo).

### 3.5 Costos y escalabilidad

- Lista de materiales de un nodo con precios y costo total.
- Costo de cubrir el campus con N nodos frente al de una campaña de medición con sonómetro profesional.
- Consumo de energía, mantenimiento y qué pasa cuando un nodo falla.

### 3.6 A quién le sirve

Nombrar usuarios concretos y qué decisión toman con el sistema:

- Bienestar Universitario: horarios y distribución de la cafetería.
- Planeación / infraestructura: dónde invertir en tratamiento acústico.
- Biblioteca y docentes: zonas y horas de silencio.
- Estudiantes: consultar en el celular dónde hay un sitio tranquilo ahora.

Una carta o un correo de interés de una de estas dependencias vale más que cualquier diapositiva.

### 3.7 Entregables de la propuesta

1. Documento: problema, objetivos medibles, arquitectura, metodología de medición, resultados preliminares, cronograma, presupuesto, riesgos.
2. Diagrama de arquitectura de una sola página: nodo → MQTT → Raspberry Pi → base de datos → panel.
3. Demostración en vivo de 3–5 minutos con este guion: el problema → el nodo físico → dato en vivo en el mapa → vista 3D del edificio → evento registrado → lo que el sistema anticipa.
4. Video de respaldo.
5. Código y documentación en un repositorio.

### 3.8 Riesgos a declarar

| Riesgo | Mitigación |
|---|---|
| Sin internet en la sala | Librerías locales, video de respaldo, modo demostración |
| El nodo falla durante la demo | Segundo nodo o datos históricos reales cargados |
| Cuestionan la precisión | Tabla de calibración y advertencia de uso indicativo |
| Cuestionan lo de "gemelo" | Predicción o escenarios (3.1) |
| Lluvia, vandalismo o robo del nodo | Caja estanca, ubicación alta, respaldo en SD |

## 4. Prioridades si el tiempo es corto

1. Fase 1: datos reales en el panel.
2. Fase 2: calibración contra sonómetro.
3. Sección 3.2: una semana de datos y la frase de impacto.
4. Sección 3.1: la predicción sobre la gráfica de 24 h.
5. Fase 3: campus trazado en 3D.

Todo lo demás mejora la propuesta, pero estos cinco puntos la sostienen.

## 5. Datos que faltan para afinar este plan

- Para qué es la propuesta: materia, semillero, convocatoria o trabajo de grado, y la fecha de entrega.
- Los criterios de evaluación, si existen.
- Cuántos nodos hay construidos y con qué micrófono.
- Qué base de datos guarda el histórico.
- Si hay acceso a un sonómetro de referencia y a un dron.
