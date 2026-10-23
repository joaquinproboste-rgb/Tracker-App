# Joaquín Coach

App para un entrenador personal y sus clientes, al estilo Gymly. Todo el texto de la interfaz va en español.

## Estado actual

Proyecto Vite + React + TypeScript que reproduce el prototipo original (`prototipo/index.html`, un solo archivo sin librerías, se conserva como referencia). Usa datos de ejemplo para 3 clientes y guarda todo en `localStorage` (clave `joaquin-coach-v1`, mismo formato que el prototipo).

- `npm run dev` levanta la app en http://localhost:5173 (el prototipo queda en `/prototipo/index.html`). `npm run build` hace el chequeo de tipos y la compilación.
- `src/lib/`: lógica sin React — `types.ts` (modelo de datos), `exercises.ts`, `seed.ts` (datos de ejemplo), `metrics.ts` (estrés, volumen, 1RM, tonelaje), `assistant.ts` (reglas), `achievements.ts`, `audio.ts`, `utils.ts`. Las funciones reciben `db` y `exm` (mapa de ejercicios) explícitos.
- `src/store.tsx`: contexto con los datos (`update(fn)` clona, aplica, guarda y devuelve la nueva versión) y el estado de la interfaz.
- `src/components/`: piezas compartidas (figura animada, gráficos SVG, escalas, tabs, insignias). `src/client/`, `src/trainer/`, `src/library/`: pantallas de cada modo.
- `src/styles.css`: el CSS del prototipo sin cambios.

### Supabase (etapa 2)
- Sin `.env.local` la app corre en **modo demo** (datos de ejemplo en `localStorage`, selector Cliente/Entrenador). Con `VITE_SUPABASE_URL` y `VITE_SUPABASE_ANON_KEY` en `.env.local` (ver `.env.example`) pide iniciar sesión y usa la base de datos. Nunca usar la clave `service_role` en el frontend.
- `supabase/schema.sql`: tablas, RLS y triggers; se ejecuta completo en el SQL Editor y es re-ejecutable. El correo del entrenador está en `trainer_emails` al final del script.
- Acceso: inicio de sesión con enlace al correo (`src/auth/`). El trigger `handle_new_user` da rol de entrenador a los correos de `trainer_emails`; un cliente queda vinculado si el entrenador lo agregó antes con ese correo (o al agregarlo después, trigger `link_client_account`). Sin invitación la cuenta no tiene acceso. El rol decide el modo; no hay selector.
- RLS: el entrenador ve y edita todo; el cliente lee su ficha, plan y biblioteca, y escribe solo sus sesiones, sesión en curso, check-ins semanales y logros. Solo el entrenador modifica planes y decisiones del asistente.
- `src/lib/backend.ts` (demo) y `src/lib/remote.ts` (Supabase) implementan `Backend.save(db)`. El remoto compara con lo último sincronizado y escribe solo las filas cambiadas; solo borra sesiones en curso y ejercicios propios, nunca historial. Los componentes no saben qué backend se usa.
- Supabase Auth → URL Configuration: la Site URL y las Redirect URLs deben incluir la dirección de la app (http://localhost:5173 en desarrollo).

### Modo cliente
- Rutina del día: ejercicio, carga, repeticiones, series y RPE objetivo por serie; el cliente registra lo real y marca cada serie.
- Check-in previo (motivación, sueño, descanso, alimentación, escala 1–5) y posterior (RPE de la sesión 1–10, sensación 1–5, comentario).
- Check-in semanal (energía, fatiga, rendimiento, comentario).
- Temporizador de descanso: parte al marcar una serie, con el descanso definido por ejercicio en el plan.
- Barra fija con tonelaje de la sesión en vivo y cuánto falta para el próximo logro.
- 14 logros (sesiones, rachas de semanas completas, toneladas por sesión y acumuladas, récords, check-ins) con celebración al desbloquearlos.
- Biblioteca de ejercicios con figuras animadas, claves técnicas y link a video.

### Modo entrenador
- Lista de clientes con índice de estrés y sugerencias pendientes; agregar cliente (nombre, objetivo, correo de invitación).
- Resumen: sesiones de la semana, volumen, índice de estrés, adherencia; gráficos de series por grupo muscular, volumen semanal, estrés, 1RM estimado, RPE real vs. planificado y check-in previo.
- Planificador: días, ejercicios, series (carga, reps, RPE) y descanso por ejercicio.
- Crear y editar ejercicios: peso corporal o implemento (barra, mancuerna, kettlebell, máquina, polea, banda elástica, disco, otro), tipo de registro (kg y reps, solo reps, solo tiempo, kg y tiempo), figura animada, link de video, claves técnicas.
- Asistente por reglas: propone descarga, subida de carga o cambio de estímulo con su justificación y vista previa de cambios. **El asistente solo recomienda; nada cambia en el plan hasta que el entrenador lo aplica.** Mantener este principio en cualquier versión futura.

### Fórmulas
- Índice de estrés (0–100) = 60% × ((5 − promedio check-in previo) / 4 × 100) + 40% × (RPE de sesión × 10). Bandas: <40 bajo, 40–60 moderado, >60 alto.
- 1RM estimado = carga × (1 + (reps + RIR) / 30), con RIR = 10 − RPE (Epley ajustado por RPE).
- Volumen / tonelaje = Σ carga × reps de series hechas, solo ejercicios "kg y reps".
- Reglas del asistente (2 últimas semanas completas): descarga si estrés ≥ 60 o (RPE real − objetivo ≥ 0,8 y sueño ≤ 2,5); subir carga si RPE real − objetivo ≤ −0,8 y estrés < 50; cambiar estímulo si el 1RM estimado de sentadilla, banca y peso muerto sube < 2% en 4 semanas.

## Siguiente etapa (lo que falta para que sea una app real)
1. ~~Pasar a un proyecto con Vite + React + TypeScript~~ (hecho, octubre 2026).
2. Backend con Supabase: código y `supabase/schema.sql` listos (octubre 2026); falta conectarlo a un proyecto real y probarlo con cuentas de entrenador y cliente.
3. PWA instalable (manifest + service worker) para usar en el celular sin conexión durante la sesión.
4. Más adelante: asistente con la API de Claude que lea las mismas métricas y converse con el entrenador; el entrenador sigue decidiendo.
