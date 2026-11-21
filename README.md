# 📦 Сборка APK в GitHub Actions

Готовый шаблон для автоматической сборки Android APK в GitHub.

## 📁 Что внутри ZIP

```
apk-github-actions/
├── .github/workflows/
│   ├── build-apk.yml      ← основной workflow для Android Native
│   ├── flutter.yml        ← для Flutter проектов
│   └── react-native.yml   ← для React Native проектов
├── app/                   ← минимальный Android проект (Kotlin)
├── gradle/wrapper/        ← Gradle Wrapper
├── build.gradle
├── settings.gradle
├── gradlew
└── README.md
```

## 🚀 Как использовать

### Вариант 1: Использовать шаблон как есть

1. Распакуй ZIP
2. Создай новый репозиторий на GitHub
3. Загрузи все файлы в репозиторий (включая скрытую папку `.github`)
4. Сделай push в ветку `main`
5. GitHub Actions автоматически соберёт APK
6. Скачай готовый APK из вкладки **Actions → Artifacts**

### Вариант 2: Добавить workflow в свой проект

Если у тебя уже есть проект:

1. Скопируй нужный файл из `.github/workflows/` в свой репозиторий:
   - Android Native → `build-apk.yml`
   - Flutter → `flutter.yml`
   - React Native → `react-native.yml`
2. Сделай commit и push
3. APK появится в **Actions → Artifacts**

> ⚠️ Важно: папка должна называться именно `.github/workflows/`

## 🔧 Что собирает workflow

- ✅ **Debug APK** — для тестов, без подписи
- ✅ **Release APK** — для публикации (пока без подписи, см. ниже)

Артефакты доступны 90 дней в вкладке Actions.

## 🔐 Подписанный Release APK (для Google Play)

Для публикации в Google Play нужен подписанный APK/AAB.

### Шаг 1: Создай keystore локально

```bash
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key
```

### Шаг 2: Добавь секреты в GitHub

В репозитории: **Settings → Secrets and variables → Actions → New repository secret**

Добавь 4 секрета:
- `KEYSTORE_BASE64` — base64 от keystore файла
- `KEYSTORE_PASSWORD` — пароль от keystore
- `KEY_ALIAS` — alias ключа
- `KEY_PASSWORD` — пароль от ключа

Получить base64:
```bash
base64 -w 0 my-release-key.jks
```

### Шаг 3: Используй workflow с подписью

Замени шаг сборки в `build-apk.yml` на:

```yaml
      - name: Decode keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > app/my-release-key.jks

      - name: Build Signed Release APK
        run: ./gradlew assembleRelease
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
```

И добавь в `app/build.gradle`:

```gradle
android {
    signingConfigs {
        release {
            storeFile file("my-release-key.jks")
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

## 📱 Для Flutter

Если твой проект на Flutter, удали папку `app/` и используй `flutter.yml`:

```bash
flutter create my_app
# скопируй .github/workflows/flutter.yml в проект
```

## ⚛️ Для React Native

Если проект на React Native, используй `react-native.yml`. Папка `android/` должна быть в корне проекта.

## 🛠 Требования

- GitHub аккаунт
- Репозиторий с кодом
- Для локальной сборки: Android Studio / JDK 17

## ❓ Частые проблемы

**Workflow не запускается** — проверь, что файл лежит в `.github/workflows/` и ветка называется `main` или `master`.

**Gradle не найден** — убедись, что в проекте есть `gradlew` и папка `gradle/wrapper/`.

**APK не скачивается** — зайди в Actions → выбери последний запуск → прокрути вниз до Artifacts.

---

Сделано для быстрой сборки APK в GitHub Actions 🚀
