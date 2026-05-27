# Instrucciones para GitHub Copilot en el proyecto Kovara Banking

## Descripción General del Proyecto

Este es un proyecto de banca en línea desarrollado con Next.js y TypeScript, utilizando el App Router. La interfaz de usuario se construye con **shadcn/ui** y **Tailwind CSS**. La gestión de formularios se realiza a través de **React Hook Form** y la validación de esquemas con **Zod**. El backend está impulsado por **Appwrite** para la gestión de usuarios y datos, y se integra con **Plaid** para la vinculación de cuentas bancarias y transacciones. El monitoreo de errores y rendimiento se realiza con **Sentry**.

## Arquitectura y Flujo de Datos

- **App Router de Next.js**: La estructura de la aplicación se basa en el App Router, con rutas y componentes definidos en el directorio `src/app`. Las rutas principales se dividen en dos grupos: `(auth)` para la autenticación (sign-in, sign-up) y `(root)` para la aplicación principal post-autenticación.
- **Componentes**: Los componentes reutilizables de la interfaz de usuario se encuentran en `src/components/ui`, siguiendo el patrón de **shadcn/ui**. Los componentes específicos de la aplicación están en `src/components` y los componentes de página se localizan junto a sus rutas correspondientes.
- **Gestión de Estado y Lógica de Backend**: El estado de los formularios se gestiona localmente con `React Hook Form`. Las interacciones con el servidor se manejan a través de **Server Actions** de Next.js, definidas en `src/lib/actions` y `src/services/actions`. Estas acciones se comunican directamente con los servicios de Appwrite y Plaid.
- **Flujo de Formulario Típico**: Un formulario de la UI (ej. `SignUpForm.tsx`) utiliza `React Hook Form` y un esquema de `Zod` (`src/schemas`) para la validación del lado del cliente. Al enviar, se invoca una Server Action (ej. `signUp` en `src/lib/actions/user.actions.ts`) que realiza la lógica de negocio en el servidor.

## Flujos de Trabajo del Desarrollador

- **Ejecutar el Servidor de Desarrollo**: Para iniciar la aplicación en modo de desarrollo, utiliza el siguiente comando:
  ```bash
  npm run dev
  ```
- **Validación de Formularios**: Los esquemas de validación se definen con **Zod** en el directorio `src/schemas`. Por ejemplo, `signUpSchema.ts` contiene las reglas para el formulario de registro.
- **Componentes de Formularios**: Utiliza los componentes personalizados como `FormFieldInput` para los campos de entrada en los formularios. Estos componentes están diseñados para integrarse con `form.control` de `React Hook Form`.

## Convenciones y Patrones Específicos del Proyecto

- **Formularios**: Todos los formularios deben usar `React Hook Form` y `zodResolver` para la validación.
- **Acciones del Servidor**: La lógica del lado del servidor, como el registro de usuarios (`signUp`) o las operaciones de Plaid (`exchangePublicToken`), se implementa como Server Actions.
- **Estilos**: Los estilos se aplican principalmente con clases de **Tailwind CSS**. Los componentes de **shadcn/ui** ya vienen con estilos predefinidos que se pueden personalizar.
- **Manejo de Errores**: Las llamadas a APIs externas (Plaid, Appwrite) deben estar envueltas en bloques `try...catch`. Los errores se capturan con `Sentry.captureException(error)`, como se muestra en `src/services/actions/plaid.ts`.

## Integraciones y Dependencias Externas

- **Appwrite**: Es el backend principal para la autenticación y la base de datos. La configuración del cliente de Appwrite se encuentra en `src/lib/server/appwrite.ts`.
- **Plaid**: Se utiliza para vincular cuentas bancarias. La lógica principal está en `src/services/actions/plaid.ts`. Consulta `.github/instructions/plaid.instructions.md` para un contexto detallado.
- **shadcn/ui**: La mayoría de los componentes de la interfaz de usuario provienen de esta biblioteca.
- **React Hook Form y Zod**: Son la base para la creación y validación de formularios.
- **Sentry**: Se utiliza para el monitoreo de errores y rendimiento. La configuración se encuentra en los archivos `sentry.*.config.ts` y `instrumentation.ts`. Consulta `.github/instructions/sentry.instructions.md` para más detalles.

## Archivos y Directorios Clave

- `src/app/(auth)/sign-up/SignUpForm.tsx`: Un excelente ejemplo de cómo se implementan los formularios.
- `src/lib/actions/user.actions.ts`: Contiene la lógica de negocio del lado del servidor para las operaciones de usuario.
- `src/services/actions/plaid.ts`: Contiene la lógica de negocio para la integración con Plaid.
- `src/lib/server/appwrite.ts`: Define la conexión y configuración del cliente de Appwrite.
- `src/lib/server/plaid.ts`: Define la conexión y configuración del cliente de Plaid.
- `src/schemas/`: Directorio con todos los esquemas de validación de Zod.
- `.github/instructions/`: Contiene guías detalladas para las integraciones clave como Plaid y Sentry.

---
> Source: [aaronportobanco/kovara-banking](https://github.com/aaronportobanco/kovara-banking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-27 -->
