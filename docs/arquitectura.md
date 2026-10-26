# Arquitectura y orquestación

El agente utiliza **LangGraph `StateGraph`** para representar explícitamente el flujo de ejecución.
LangChain aporta las herramientas y mensajes; Mistral se utiliza para generación, clasificación,
planificación y embeddings; FAISS soporta recuperación semántica del corpus y de la memoria persistente.

```mermaid
flowchart TD
    U[Usuario] --> P[Preparar contexto]
    P --> R{Router LangGraph}

    R -->|CONSULTA| C[Nodo Consulta]
    R -->|GESTION| G[Nodo Gestión]
    R -->|FUERA| F[Respuesta controlada]

    C --> CT[Tools de consulta]
    CT --> RAG[RAG: Mistral Embed + FAISS]
    CT --> ML[Tool recordar / memoria persistente]
    RAG --> C
    ML --> C

    G --> PL[Planificador]
    PL --> TW[Tools de validación y escritura]
    TW --> O{Observación}

    O -->|OK| FIN[Finalizar]
    O -->|Faltan datos| TP[Persistir tarea pendiente]
    O -->|Rechazo reintentable| RP[Replanificar]
    O -->|Restricción definitiva| FIN

    RP --> PL
    TP --> FIN
    C --> FIN
    F --> FIN

    FIN --> MC[Memoria de corto plazo]
    FIN --> MLP[Memoria de largo plazo]
    FIN --> RESP[Respuesta + trazabilidad]
```

## Componentes

- **Router:** clasifica cada solicitud en `CONSULTA`, `GESTION` o `FUERA`.
- **Consulta:** ejecuta herramientas de lectura, RAG y memoria cuando corresponde.
- **Gestión:** aplica Plan-and-Execute, valida datos, ejecuta tools de escritura y replantea ante condiciones reintentables.
- **Tarea pendiente:** persiste una tarea estructurada cuando faltan datos y permite retomarla en un turno posterior.
- **Memoria de corto plazo:** conserva hasta tres turnos recientes.
- **Memoria de largo plazo:** FAISS persistente con prioridad para estados finales verificados.
- **Guardrails:** separación `INT-*` / `OP-*`, control de datos sensibles y rechazo de consultas fuera de alcance.
