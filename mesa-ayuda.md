# 🛠️ Sistema de Mesa de Ayuda (Help Desk) - Análisis y Diseño

Este repositorio contiene el relevamiento de requisitos, casos de uso y modelado UML para un sistema de gestión de tickets de soporte técnico. El objetivo del proyecto es traducir las necesidades operativas de un área de soporte en especificaciones técnicas claras mediante diagramas como código (**Mermaid.js**).

---

## 📋 1. Relevamiento de Requisitos

### Requisitos Funcionales (RF)
* **RF01 - Creación de Tickets:** El usuario cliente debe poder registrar un nuevo incidente detallando asunto, descripción y nivel de urgencia.
* **RF02 - Asignación de Técnicos:** El supervisor o sistema debe poder asignar un ticket abierto a un técnico disponible según su especialidad.
* **RF03 - Seguimiento y Comentarios:** Tanto el cliente como el técnico pueden añadir comentarios o adjuntar capturas al historial del ticket.
* **RF04 - Cambio de Estado:** El técnico puede actualizar el estado del ticket (`Abierto`, `En Proceso`, `Resuelto`, `Cerrado`).
* **RF05 - Encuesta de Satisfacción:** Una vez resuelto el ticket, el cliente puede calificar la atención recibida para cerrar definitivamente el caso.

### Requisitos No Funcionales (RNF)
* **RNF01 - Usabilidad:** La interfaz de carga de tickets no debe requerir más de 3 pasos para el usuario final.
* **RNF02 - Tiempos de Respuesta (SLA):** El sistema debe emitir una alerta automática si un ticket de prioridad "Alta" pasa más de 2 horas sin ser asignado.
* **RNF03 - Trazabilidad:** Todo cambio de estado debe quedar registrado con fecha, hora y usuario responsable (Auditoría).

---

## 👥 2. Diagrama de Casos de Uso ( Flujo de Actores )

Representación de las interacciones principales entre los tres actores del sistema (**Cliente**, **Técnico de Soporte** y **Supervisor**) y los casos de uso identificados:

```mermaid
flowchart LR
    %% Actores
    Cliente([👤 Cliente])
    Tecnico([🛠️ Técnico de Soporte])
    Supervisor([👔 Supervisor])

    %% Subsistema
    subgraph Sistema de Mesa de Ayuda
        CU1(Crear Ticket de Soporte)
        CU2(Consultar Estado de Ticket)
        CU3(Agregar Comentario / Evidencia)
        CU4(Asignar Ticket a Técnico)
        CU5(Cambiar Estado del Ticket)
        CU6(Registrar Solución Técnica)
        CU7(Calificar Atención y Cerrar)
        CU8(Monitorear Métricas y SLA)
    end

    %% Relaciones Cliente
    Cliente --> CU1
    Cliente --> CU2
    Cliente --> CU3
    Cliente --> CU7

    %% Relaciones Técnico
    Tecnico --> CU2
    Tecnico --> CU3
    Tecnico --> CU5
    Tecnico --> CU6

    %% Relaciones Supervisor
    Supervisor --> CU4
    Supervisor --> CU8
    CU5 -.->|<<include>>| CU6
```

---

## 🔄 3. Ciclo de Vida del Ticket (Diagrama de Estados UML)

Modelado del comportamiento de un ticket desde que es reportado hasta su cierre definitivo:

```mermaid
stateDiagram-v2
    [*] --> Abierto : Cliente crea el ticket (RF01)
    
    Abierto --> Asignado : Supervisor asigna técnico (RF02)
    Asignado --> En_Proceso : Técnico inicia diagnóstico (RF04)
    
    En_Proceso --> Esperando_Cliente : Requiere más datos (RF03)
    Esperando_Cliente --> En_Proceso : Cliente responde
    
    En_Proceso --> Resuelto : Técnico aplica solución
    
    Resuelto --> En_Proceso : Cliente rechaza solución (Reapertura)
    Resuelto --> Cerrado : Cliente confirma y califica (RF05)
    
    Cerrado --> [*]
```

---

## 🗂️ 4. Modelo de Dominio (Diagrama de Clases UML)

Estructura de las entidades principales para la futura base de datos y lógica de negocio:

```mermaid
classDiagram
    class Usuario {
        +int id
        +String nombre
        +String email
        +String rol
    }

    class Ticket {
        +int nroTicket
        +String asunto
        +String descripcion
        +Date fechaCreacion
        +String prioridad
        +String estado
        +cambiarEstado(nuevoEstado)
        +asignarTecnico(Tecnico)
    }

    class Comentario {
        +int id
        +String texto
        +Date fechaHora
    }

    class Categoria {
        +int id
        +String nombre
        +int slaHoras
    }

    Usuario "1" --> "0..*" Ticket : reporta / atiende
    Ticket "1" *-- "0..*" Comentario : contiene
    Categoria "1" <-- "0..*" Ticket : clasifica
```

---

## 📌 Especificación Detallada: Caso de Uso Principal (CU01)

| Campo | Descripción |
| :--- | :--- |
| **Nombre** | **CU01 - Crear Ticket de Soporte** |
| **Actor Principal** | Cliente (Usuario interno o externo) |
| **Precondiciones** | El usuario debe haber iniciado sesión en el portal de autogestión. |
| **Flujo Normal** | 1. El cliente selecciona "Nuevo Ticket".<br>2. El sistema muestra el formulario con Categoría, Prioridad, Asunto y Descripción.<br>3. El cliente completa los datos y envía el formulario.<br>4. El sistema valida los campos, asigna un número único, establece el estado en `Abierto` y notifica al área de soporte. |
| **Flujo Alternativo** | **3a. Datos incompletos:** El sistema resalta los campos obligatorios faltantes y solicita completarlos antes de guardar. |
| **Postcondiciones** | El ticket queda registrado en el sistema y pendiente de asignación en el panel de soporte. |