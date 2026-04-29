```markdown
# Require Compliant Device and MFA for Office 365

## Descripción General
Este proyecto documenta la implementación y validación de una política de acceso condicional que exige **dispositivo compatible (compliant)** Y **autenticación multifactor (MFA)** para acceder a Office 365.

La política fue diseñada como parte de un proyecto de consultoría de seguridad para V-Ready Group LLC y validada en un entorno de laboratorio.

## Política Implementada
- **Nombre en Azure:** `Grant - Require Compliant Device and MFA for Office 365 - SG-Admins-Practica`
- **Propósito:** Añadir una doble capa de seguridad: solo los usuarios con un dispositivo corporativo que cumpla las políticas de seguridad (gestionado por Intune) y que verifiquen su identidad con MFA pueden acceder a los recursos de Office 365.
- **Asignación:** Grupo piloto `SG-Admins-Practica`.
- **Recurso Objetivo:** `Office 365`.
- **Condiciones:** Cualquier plataforma de dispositivo (`Any device`).
- **Controles de Concesión:** `Grant access` + `Require multifactor authentication` + `Require device to be marked as compliant`. Se requiere el cumplimiento de **todos** los controles.
- **Estado:** `Report-only` (Solo informe) para validación.

### Captura de la Política
*[Agrega aquí una captura de pantalla de la política en Azure: `img/policy-overview.png`]*

## Resultado de la validación con "What If"
La política se evaluó con la herramienta "What If", simulando un acceso desde un dispositivo Windows en navegador. El resultado confirmó que la política se aplicaría, exigiendo simultáneamente MFA y dispositivo compatible.

### Captura del Resultado de "What If"
*[Agrega aquí una captura de pantalla del resultado de la simulación: `img/whatif-result.png`]*

## Diagnóstico de Error Real (Simulado)
Durante la fase de pruebas, se simuló el acceso de un usuario sin dispositivo compatible. Aunque la política está en modo `Report-only` y no bloqueó el acceso, se documentó el mensaje de error que aparecería si la política estuviera activa en modo `On`. Este mensaje está documentado en los foros oficiales de Microsoft como parte del comportamiento esperado ante una política de acceso condicional no satisfecha: *"Your sign-in was successful but does not meet the criteria to access this resource. For example, you might be signing in from a browser, app, location, or an authentication flow that is restricted by your admin."*.

### Mensaje de Error Esperado (Cuando la política esté "On")
Si un usuario sin dispositivo conforme intenta acceder a Office 365, verá el mensaje indicado anteriormente.

### Flujo de Diagnóstico en Sign-in Logs
*[Agrega aquí una captura de pantalla del Sign-in log mostrando la pestaña de Conditional Access: `img/signin-log-error.png`]*

**Tip de Experto:** En la captura de los Sign-in Logs, asegúrate de resaltar el **Correlation ID**. En consultoría real, ese identificador es la clave para rastrear qué ocurrió exactamente en los servidores de Microsoft. Incluirlo en un reporte de este tipo le aporta un enfoque de investigación forense muy profesional.

## Pasos de Remediación
Para resolver el bloqueo y permitir el acceso, el usuario o el administrador de TI deben seguir estos pasos:

1.  **Unir el dispositivo a Azure AD (Entra ID):**
    *   En Windows 10/11, ir a `Settings > Accounts > Access Work or School > Connect` y seguir los pasos para unir el dispositivo.
    *   Algunos entornos requieren que el usuario tenga privilegios de administrador local para completar la unión.

2.  **Verificar la inscripción automática en Microsoft Intune:**
    *   La unión a Azure AD permite la inscripción automática en Intune, siempre que el administrador la haya configurado previamente en **Microsoft Entra ID > Mobility (MDM and MAM)** con el scope de usuarios adecuado.
    *   Si un dispositivo ya estaba unido a Azure AD antes de configurar la inscripción automática, será necesario desunirlo y volverlo a unir para forzar la inscripción.

3.  **Comprobar el cumplimiento del dispositivo:**
    *   El estado de cumplimiento se puede verificar directamente desde el dispositivo en `Settings > Accounts > Access Work or School`, seleccionando la cuenta corporativa y haciendo clic en **"Info"**.
    *   En el **Microsoft Intune admin center**, el administrador puede confirmar el cumplimiento revisando el dispositivo en `Devices > All devices`.

4.  **Forzar la sincronización si es necesario:**
    *   Si el dispositivo se muestra conforme en Intune pero el acceso condicional sigue bloqueando, se puede forzar una sincronización manual desde `Settings > Accounts > Access Work or School`, seleccionando la cuenta y haciendo clic en **"Sync"**.
    *   El estado de cumplimiento puede tardar unos minutos en reflejarse tras la sincronización.

## Hallazgos Clave del Laboratorio
*   **El modo `Report-only` no aplica controles:** Tras aislar las demás políticas, se confirmó que un usuario puede acceder sin MFA ni dispositivo compatible si la única política activa está en este modo. Los resultados solo se registran en los logs.
*   **Mensaje de error estándar:** El mensaje *"Your sign-in was successful but does not meet the criteria to access this resource..."* es el mensaje genérico que verá el usuario cuando la política esté en modo `On`.
*   **Importancia de la Limpieza de Políticas:** La interacción de múltiples políticas en un mismo grupo puede generar bloqueos inesperados difíciles de diagnosticar. Se recomienda un diseño con exclusiones claras y pilotos aislados.

## Observaciones de Arquitecto para Producción
Antes de migrar esta política a modo "On", se deben considerar los siguientes puntos críticos:

### 1. Cuentas de Emergencia (Break-glass)
- Se deben crear al menos dos cuentas de administrador de emergencia (ej. `breakglass1@dominio.onmicrosoft.com`) y **excluirlas explícitamente** de todas las políticas de acceso condicional.
- Estas cuentas garantizan el acceso al portal de administración en caso de un fallo global de Intune o de la propia política, permitiendo su desactivación.

### 2. Filtrado de Plataformas de Dispositivo
- La política actual se configuró con **"Any device"** como prueba. En un despliegue real, se recomienda acotarla a las plataformas que estén efectivamente gestionadas por Intune (ej. **Windows** y, si aplica, **macOS** o **iOS**).
- Un administrador que intente acceder desde un sistema no soportado (como Linux) quedará bloqueado de forma permanente hasta que use un dispositivo compatible.

### 3. Requisito de Navegador para Detección de Cumplimiento
- Para que el estado de cumplimiento del dispositivo sea correctamente evaluado en el navegador al acceder a Office 365, es necesario:
  - **Microsoft Edge:** La detección es nativa si el dispositivo está unido a Azure AD.
  - **Google Chrome:** El usuario debe instalar la extensión oficial **"Windows Accounts"** (o "Microsoft Accounts").
- La falta de esta configuración hará que la política deniegue el acceso aunque el dispositivo esté conforme en Intune.

### 4. Manejo de Dispositivos Existentes
- Los dispositivos que ya estaban unidos a Azure AD antes de habilitar la inscripción automática en Intune **no se inscribirán automáticamente**.
- Será necesario desunirlos y volverlos a unir para forzar su inscripción y que pasen a ser gestionados.

## Próximos Pasos
*   Monitorear los resultados en modo `Report-only` durante un ciclo de prueba.
*   Activar la política en modo **"On"** para un subconjunto de usuarios con dispositivos ya gestionados por Intune.
*   Comunicar el cambio a los usuarios piloto y proporcionar instrucciones claras para unir sus dispositivos.
*   Asegurar que al menos un grupo de prueba disponga de dispositivos Windows corporativos unidos a Entra ID y gestionados por Intune.

---
*Proyecto de consultoría interna para V-Ready Group LLC | Documentado por Jorge Vazquez*
```
