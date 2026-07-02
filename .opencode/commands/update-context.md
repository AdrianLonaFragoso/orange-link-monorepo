---
description: Escanea ambos proyectos de Orange Link y actualiza AGENTS.md con la estructura actualizada (páginas, rutas API, modelos BD, etc.).
---

Escanea exhaustivamente los directorios `orange-link-app/` y `orange-link-back/` para detectar cambios. Sigue estos pasos:

1. Revisa `orange-link-app/src/components/screens/` y lista todas las pantallas existentes (nombres de archivo + propósito inferido de su código)
2. Revisa `orange-link-app/src/lib/api.ts` para listar los endpoints del API client
3. Revisa `orange-link-app/src/hooks/useAppStore.ts` para entender el estado global
4. Revisa `orange-link-back/src/routes/` para listar todas las rutas API
5. Revisa `orange-link-back/prisma/schema.prisma` para los modelos de BD
6. Actualiza `AGENTS.md` con la información actualizada, manteniendo el mismo formato y estructura de secciones

NO borres secciones existentes a menos que el feature ya no exista. Añade nuevas secciones si hay nuevos patrones, configuraciones o dependencias relevantes.
