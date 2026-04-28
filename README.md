# 🔐 Entra Conditional Access Baseline

## 📌 Descripción General
Este proyecto documenta el diseño, implementación y validación de un conjunto de políticas de seguridad fundamentales (baseline) utilizando **Microsoft Entra Conditional Access**.  
Las políticas reflejan un estándar mínimo de protección para entornos empresariales modernos, siguiendo las mejores prácticas recomendadas por Microsoft y los principios de **Zero Trust**.

---

## 🛡️ Políticas Implementadas
Todas las políticas han sido creadas en modo **"Report-only"** (Solo informe) para su evaluación y auditoría, previo paso a producción.

### 1️⃣ Bloquear Acceso desde Fuera de México
- **Nombre en Azure:** `Block - Non-Mexico Access - SG-Admins-Practica`
- **Propósito:** Restringir el acceso a los recursos corporativos exclusivamente a conexiones originadas en territorio nacional.
- **Asignación:** Grupo piloto `SG-Admins-Practica`.
- **Condición de Ubicación:** Excluye la ubicación con nombre `México` y aplica la política a cualquier otra ubicación.
- **Control de Acceso:** `Block access` (Bloquear).
- **Validación (What If):** Simulación con IP de Brasil (177.54.148.0). La política se aplicó correctamente.

### 2️⃣ Exigir MFA para el Portal de Azure
- **Nombre en Azure:** `Grant - MFA for Azure Portal - SG-Admins-Practica`
- **Propósito:** Añadir una capa adicional de verificación multifactor para proteger el acceso a la consola de administración de Azure.
- **Asignación:** Grupo piloto `SG-Admins-Practica`.
- **Recurso Objetivo:** `All cloud apps` (alcance temporal para laboratorio).
- **Control de Acceso:** `Grant access` + `Require multifactor authentication`.
- **Validación (What If):** Confirmado que la política exige MFA para usuarios del grupo en cualquier aplicación, incluida la gestión de Azure.

### 3️⃣ Bloquear Protocolos de Autenticación Legacy
- **Nombre en Azure:** `Block - Legacy Authentication - All Users`
- **Propósito:** Eliminar la superficie de ataque bloqueando protocolos inseguros como POP3, IMAP y Exchange ActiveSync.
- **Asignación:** `All users` (excluyendo la cuenta de administrador para evitar bloqueos accidentales).
- **Condición de Cliente:** `Exchange ActiveSync clients` + `Other clients`.
- **Control de Acceso:** `Block access` (Bloquear).
- **Validación (What If):** Simulación con cliente legacy (Other clients) y aplicación Exchange Online. La política bloqueó el acceso correctamente.

---

## 🧪 Herramientas de Verificación
- **What If:** Se utilizó para simular escenarios reales (IP extranjera, cliente legacy) y confirmar el comportamiento esperado de las políticas.
- **Named Locations:** Se definió la ubicación `México` como referencia geográfica para la primera política.

---

## 📈 Próximos Pasos
- Monitorear los resultados en modo "Report-only" durante un ciclo de prueba.
- Ajustar exclusiones y condiciones según los registros de inicio de sesión.
- Activar las políticas en modo **"On"** una vez validada su eficacia.
- Integrar estas políticas con **Identity Protection** y **Privileged Identity Management (PIM)** para una estrategia completa de defensa en profundidad.

---

## 👤 Autor
**Jorge Vazquez**  
*Identity & Cybersecurity Architect*  
[LinkedIn](https://www.linkedin.com/in/tu-perfil) | [GitHub](https://github.com/tu-usuario)
