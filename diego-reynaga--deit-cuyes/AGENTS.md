<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Instrucciones para GitHub Copilot - Sistema de Gestión de Cuyes INIA

## Contexto del Proyecto
Este es un sistema web Flask para la gestión y venta de cuyes en el INIA de Andahuaylas. El sistema sigue el patrón MVC y está diseñado para usuarios con conocimientos básicos de computación.

## Pautas de Desarrollo

### Arquitectura
- Seguir estrictamente el patrón MVC (Modelo-Vista-Controlador)
- Usar Flask-SQLAlchemy para modelos de base de datos
- Implementar blueprints para organizar las rutas
- Mantener separación clara entre lógica de negocio y presentación

### Base de Datos
- Usar MySQL como motor de base de datos
- Implementar relaciones apropiadas entre tablas
- Incluir validaciones a nivel de modelo
- Usar migraciones de Flask-Migrate para cambios de esquema

### Interfaz de Usuario
- Diseño minimalista y moderno
- Interfaz intuitiva para usuarios no técnicos
- Responsive design compatible con móviles
- Usar Bootstrap para consistencia visual
- Incluir iconos descriptivos y ayudas visuales

### Seguridad
- Implementar autenticación robusta con Flask-Login
- Usar hash de contraseñas con Flask-Bcrypt
- Validación de formularios con Flask-WTF
- Control de acceso basado en roles

### Funcionalidades Específicas
- Sistema de roles: admin, empleado, cliente
- Gestión completa de cuyes (CRUD)
- Control sanitario y tratamientos
- Sistema de ventas en línea
- Generación de reportes PDF/Excel
- Dashboard con métricas importantes

### Estilo de Código
- Usar nombres descriptivos en español para variables relacionadas con el dominio
- Comentarios en español para lógica de negocio
- Seguir PEP 8 para convenciones de Python
- Validar datos tanto en frontend como backend
- Manejar errores de forma amigable para el usuario

### Consideraciones Especiales
- Los usuarios finales tienen conocimientos básicos de computación
- Priorizar usabilidad sobre funcionalidades avanzadas
- Incluir mensajes de ayuda y confirmación
- Implementar búsquedas y filtros sencillos
- Generar reportes claros y comprensibles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diego-reynaga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
