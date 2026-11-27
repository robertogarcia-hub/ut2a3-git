# Investigación: Recuperación de configuración en `servidor.conf`

## Datos del técnico
- **Nombre y apellidos:** Roberto Garcia Brito
- **Correo:** robertogarcia230688@gmail.com
- **Repositorio:** `ut2a3-git`
- **Archivo auditado:** `servidor.conf`

## Problema detectado
El cortafuegos del servidor web (`web01`) quedó desactivado (`firewall=no` en la sección `[seguridad]`), lo que suponía un riesgo de seguridad tras varias modificaciones de configuración.

## Investigación del historial (Apartado 5)
- **Comandos utilizados:**
  - `git log --oneline` para listar los commits de forma resumida y localizar el orden de los cambios.
  - `git show 7b31736` para inspeccionar los detalles y líneas modificadas en el commit sospechoso (`-firewall=si` y `+firewall=no`).
  - `git diff 9b7fb54 7b31736` para comparar el estado anterior y posterior al cambio de seguridad.
- **Commit que introdujo el error:** `7b31736` (`Actualizacion de seguridad`).
- **Cómo se localizó:** al revisar `git log --oneline`, el commit `Actualizacion de seguridad` (`7b31736`) destacaba sobre los cambios de puerto, página principal y límite de conexiones. Al ejecutar `git show 7b31736` y `git diff 9b7fb54 7b31736`, se confirmó que cambiaba `firewall=si` por `firewall=no`.

## Recuperación temporal con `checkout` (Apartado 6)
Se ejecutó `git checkout 9b7fb54 -- servidor.conf` (junto con `cat servidor.conf`, `git status` y `git diff --cached`).

1. **¿Qué versión del archivo has recuperado?**
   La versión de `servidor.conf` correspondiente al commit `9b7fb54` (`Cambio de puerto`), justo antes de que se desactivara el cortafuegos (`puerto=8080`, `firewall=si` e `index=index.html`, sin `max_conexiones=100`).
2. **¿Se han perdido temporalmente las modificaciones posteriores?**
   Sí, en el directorio de trabajo y en el área de preparación (`staged`) el archivo volvió al estado de `9b7fb54`, por lo que los cambios posteriores (`firewall=no`, `index=inicio.html` y `max_conexiones=100`) no estaban presentes en ese momento.
3. **¿Han desaparecido los commits posteriores del historial?**
   No, el historial de commits (`git log`) sigue intacto; recuperar una versión de un archivo con `git checkout <commit> -- <archivo>` no borra los commits posteriores.
4. **¿Por qué no conviene utilizar directamente esta versión antigua como solución definitiva?**
   Porque al recuperar la versión antigua se perderían modificaciones válidas realizadas después (como `index=inicio.html` y `max_conexiones=100`), además de que la corrección debe quedar registrada como un cambio nuevo y explícito sobre la configuración actual.

## Corrección y solución definitiva (Apartado 7)
- **Cambios válidos conservados:**
  - Puerto `8080` (`puerto=8080`).
  - Página principal `inicio.html` (`index=inicio.html`).
  - Límite de 100 conexiones (`max_conexiones=100`).
- **Solución aplicada:** primero se restauró la versión más reciente con `git restore --source=HEAD --staged --worktree servidor.conf`, después se modificó únicamente `firewall=no` a `firewall=si` en `[seguridad]` y se registró el commit `"Correccion de configuracion de seguridad"`.

## Utilidad de Git para administrar archivos de configuración
Git permite llevar un registro exacto de quién, cuándo y qué línea modificó en cada archivo de configuración, comparar versiones (`git diff` y `git show`), recuperar estados anteriores ante fallos y aplicar correcciones limpias sin perder cambios válidos.