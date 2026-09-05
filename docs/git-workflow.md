# Chapai

Aplicación móvil desarrollada con React Native.

Chapai se mantiene independiente del entorno de desarrollo utilizado para compilarlo. El código fuente permanece en el host y actualmente utiliza un contenedor Docker externo para proporcionar Node.js, Java, Android SDK, NDK y CMake.

Esto permite sustituir en el futuro el entorno temporal `RN86Container` por `RNDevWorkspace` sin regenerar ni migrar el código fuente de la aplicación.

---

## 1. Stack del proyecto

| Componente         | Versión         |
| ------------------ | --------------- |
| React Native       | 0.86.3          |
| React              | 19.2.3          |
| Node.js            | 22.14.0         |
| Java               | OpenJDK 17.0.19 |
| Gradle Wrapper     | 9.3.1           |
| Android minSdk     | 24              |
| Android compileSdk | 36              |
| Android targetSdk  | 36              |
| Android NDK        | 27.1.12297006   |
| CMake              | 3.30.5          |

La versión de Gradle pertenece al propio proyecto y se gestiona mediante:

```text
android/gradle/wrapper/gradle-wrapper.properties
```

No se requiere una instalación global de Gradle.

---

## 2. Estructura del proyecto

```text
Chapai/
├── android/
├── ios/
├── __tests__/
├── App.tsx
├── index.js
├── package.json
├── package-lock.json
├── metro.config.js
├── babel.config.js
├── tsconfig.json
├── .gitignore
├── README.md
└── docs/
    └── git-workflow.md
```

---

## 3. Entorno de desarrollo

Actualmente Chapai utiliza un entorno Docker externo:

```text
PROYECTOS/
├── RN86Container/
│   └── compose.yml
│
└── ReactNativeApps/
    └── Chapai/
```

El repositorio Chapai no contiene la configuración Docker.

Dentro del contenedor, el proyecto se monta como:

```text
/workspace/Chapai
```

Mientras que físicamente permanece en el host:

```text
PROYECTOS/ReactNativeApps/Chapai
```

Esto significa que eliminar o recrear el contenedor no elimina el código fuente.

---

## 4. Preparar el entorno

El entorno Docker debe estar disponible mediante el repositorio `RN86Container`.

Desde el host:

```bash
cd ~/PROYECTOS/RN86Container
docker compose up -d
```

Verificar:

```bash
docker compose ps
```

Entrar al contenedor:

```bash
docker compose exec rn86 bash
```

Dentro:

```bash
cd /workspace/Chapai
```

---

## 5. Instalar dependencias

Después de clonar Chapai:

```bash
cd /workspace/Chapai
npm ci
```

Se recomienda `npm ci` porque respeta exactamente las versiones definidas en:

```text
package-lock.json
```

No utilizar `npm install` para instalaciones reproducibles salvo que se quiera modificar deliberadamente el árbol de dependencias.

---

## 6. Verificar React Native

```bash
node -p "require('./package.json').dependencies['react-native']"
```

Resultado esperado:

```text
0.86.3
```

React:

```bash
node -p "require('./package.json').dependencies.react"
```

Resultado esperado:

```text
19.2.3
```

---

## 7. Gradle

El proyecto utiliza únicamente Gradle Wrapper.

```bash
cd /workspace/Chapai/android
./gradlew --version
```

No utilizar:

```bash
gradle
```

global.

Para listar tareas:

```bash
./gradlew tasks
```

---

## 8. Compilar Android

Desde:

```bash
cd /workspace/Chapai/android
```

Ejecutar:

```bash
./gradlew assembleDebug
```

El APK de depuración se genera normalmente en:

```text
android/app/build/outputs/apk/debug/
```

---

## 9. Metro

Desde el contenedor:

```bash
cd /workspace/Chapai
npm start -- --host 0.0.0.0
```

Metro utiliza el puerto:

```text
8081
```

El puerto se publica mediante `RN86Container`.

---

## 10. Configuración Android

La configuración actualmente generada por React Native es:

```text
minSdkVersion     = 24
compileSdkVersion = 36
targetSdkVersion  = 36
ndkVersion        = 27.1.12297006
```

Estas configuraciones deben mantenerse en los archivos propios del proyecto Android.

No hardcodear rutas del entorno Docker como:

```text
/opt/android
```

dentro del código versionado.

Las rutas locales del Android SDK deben resolverse mediante variables de entorno o archivos locales no versionados.

---

## 11. Archivos que no deben versionarse

Nunca subir:

```text
node_modules/
.env
android/local.properties
android/.gradle/
android/app/build/
*.keystore
```

excepto archivos explícitamente permitidos por el `.gitignore`, como el keystore de debug generado por React Native cuando corresponda.

Las variables de entorno sensibles deben mantenerse fuera del repositorio.

Cuando sea necesario documentarlas, utilizar:

```text
.env.example
```

sin secretos reales.

---

## 12. Flujo Git

La guía completa se encuentra en:

```text
docs/git-workflow.md
```

Flujo básico:

```bash
git switch main
git pull --rebase origin main
git status
git diff
git add .
git commit -m "feat: descripción del cambio"
git push origin main
```

Para funcionalidades nuevas se recomienda trabajar mediante ramas:

```bash
git switch main
git pull --rebase origin main
git switch -c feat/nombre-funcionalidad
```

---

## 13. Convención de commits

Se utiliza una convención basada en Conventional Commits:

```text
feat: nueva funcionalidad
fix: corrección
docs: documentación
refactor: reorganización de código
test: pruebas
chore: mantenimiento o configuración
```

Ejemplos:

```bash
git commit -m "feat: add image capture flow"
git commit -m "fix: correct Android build configuration"
git commit -m "docs: update development environment"
git commit -m "refactor: reorganize prediction service"
git commit -m "test: add classification tests"
```

---

## 14. Versionado

Chapai utiliza Semantic Versioning:

```text
MAJOR.MINOR.PATCH
```

Ejemplos:

```text
v0.1.0
v0.2.0
v1.0.0
```

Crear un tag:

```bash
git tag -a v0.1.0 -m "Initial Chapai React Native 0.86.3 baseline"
git push origin v0.1.0
```

---

## 15. Actualizar el proyecto

Desde el host:

```bash
cd ~/PROYECTOS/ReactNativeApps/Chapai
git pull --rebase origin main
```

Si cambia `package-lock.json`:

```bash
cd ~/PROYECTOS/RN86Container

docker compose exec rn86 \
  sh -lc 'cd /workspace/Chapai && npm ci'
```

Si existen cambios Android significativos:

```bash
docker compose exec rn86 \
  sh -lc 'cd /workspace/Chapai/android && ./gradlew clean'
```

---

## 16. Migración futura a RNDevWorkspace

Chapai está diseñado para mantenerse independiente del contenedor temporal.

La transición prevista es:

```text
RN86Container
      ↓
   Chapai
      ↓
RNDevWorkspace
```

Cuando RNDevWorkspace esté disponible, Chapai no deberá regenerarse.

El objetivo será que RNDevWorkspace adopte el proyecto existente, detecte sus requisitos y prepare el runtime correspondiente.

El repositorio seguirá conservando:

```text
Chapai/
├── android/
├── ios/
├── package.json
├── package-lock.json
└── código fuente
```

y únicamente cambiará la infraestructura que proporciona las toolchains.

---

# docs/git-workflow.md

## Flujo Git de Chapai

### Actualizar `main`

```bash
git switch main
git pull --rebase origin main
```

### Revisar cambios

```bash
git status
git diff
```

### Agregar cambios

```bash
git add .
git status
```

### Crear commit

```bash
git commit -m "feat: descripción del cambio"
```

### Subir

```bash
git push origin main
```

---

## Trabajo mediante ramas

### Crear rama

```bash
git switch main
git pull --rebase origin main
git switch -c feat/nombre-funcionalidad
```

Ejemplos:

```text
feat/camera
feat/classification
feat/navigation
fix/android-build
docs/development-guide
refactor/prediction-service
test/classification
```

### Crear commits

```bash
git add .
git commit -m "feat: add camera permissions"
```

Es preferible realizar varios commits pequeños y coherentes antes que un único commit demasiado grande.

### Publicar la rama

```bash
git push -u origin feat/camera
```

Después:

```bash
git push
```

### Actualizar la rama desde `main`

```bash
git fetch origin
git rebase origin/main
```

Si existen conflictos:

```bash
git status
```

Resolverlos y:

```bash
git add archivo-resuelto
git rebase --continue
```

Si la rama ya estaba publicada:

```bash
git push --force-with-lease
```

Nunca utilizar `--force` cuando `--force-with-lease` sea suficiente.

### Después del Pull Request

```bash
git switch main
git pull --rebase origin main
git branch -d feat/camera
```

Si la rama remota continúa existiendo:

```bash
git push origin --delete feat/camera
```

---

## Convención de ramas

```text
feat/*
fix/*
docs/*
refactor/*
test/*
chore/*
```

---

## Flujo recomendado

```text
main actualizado
      ↓
crear rama
      ↓
realizar cambios
      ↓
commits pequeños
      ↓
push de rama
      ↓
Pull Request
      ↓
revisión y tests
      ↓
merge
      ↓
actualizar main local
      ↓
eliminar rama
```
