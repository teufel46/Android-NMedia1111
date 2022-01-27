## Задача Code Like a Pro

Для выполнения домашнего задания вам потребуется [скачать и установить Android Studio](https://github.com/netology-code/guides/blob/master/android/android_studio/instruction1.md).

### Легенда

Мы будем вас сразу учить работать профессионально и современно.

Что это значит? Это значит, мы собираемся на протяжении всего курса использовать самые современные инструменты и подходы.

Первый и ключевой - CI/CD. Из курса по Kotlin вы уже знаете, что это.

Для этой цели мы будем использовать [GitHub Actions](https://github.com/features/actions) и собирать ваши приложения (а потом и тестировать) при каждом пуше.


Поэтому мы сразу начнём с этого.

<details>
<summary>GitHub Actions и Actions</summary>

GitHub Actions устроен следующим образом: по наступлению определённых событий запускают worker'ы (будем считать, что это машинки с установленной ОС и ПО), в которых вы можете производить определённые операции (например, собирать код, запускать автотесты и т.д.).

Для некоторых операций есть уже готовые Actions - т.е. готовые "скрипты", которые автоматизируют часть работ в рамках GitHub Actions.

Например:
1. "Checkout" (или клонирование) репозитория в worker
1. Публикация файлов из worker'а

За клонирование отвечает [Checkout](https://github.com/marketplace/actions/checkout), а за публикацию - [Upload a Build Artifact](https://github.com/marketplace/actions/upload-a-build-artifact). Их мы и будем использовать.

Они описываются в yaml-файле в формате:
```yaml
- name: Имя шага
  uses: actions/checkout@v2 # или actions/upload-artifact@v2
  with:
    # набор опций, специфичный для конкретного Action'а
```
</details>

<details>
<summary>APK</summary>

APK (Android Package) - это файл с расширением `.apk`, в который собирается приложение для дальнейшего распространения: Google Play или установки вручную. И, конечно же, получить мы его можем с помощью инструментов, входящих в Android SDK.

Получив apk-файл, его можно простым Drag-and-Drop'ом перенести в окошко эмулятора, установив для использования.

Таким образом, наша цель - получить этот самый apk-файл. Как это сделать - читайте в разделе про Gradle Wrapper и Android Build.
</details>

<details>
<summary>Gradle Wrapper & Android Build</summary>

Мы уже знакомы с Gradle по лекциям Kotlin. Gradle - это инструмент управления проектом.

В рамках Gradle определяются задачи, которые можно выполнять с кодом проекта:
* сборка
* тестирование
* и т.д.

Gradle - это отдельный инструмент и его необходимо отдельно устанавливать.

Но чтобы с этим не "заморачиваться", сделали следующую вещь: [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) - это специальный скрипт, который поставляется вместе с вашим проектом и сам при необходимости скачивает Gradle и запускает его.

Этот скрипт расположен в файле `gradlew` (Linux/Mac) и `gradlew.bat` (Windows).

Когда вы запускаете `gradlew build`, скрипт проверяет, скачан ли Gradle. Если нет, то скачивает, а потом сам вызывает Gradle.

Есть единственный нюанс: иногда файл `gradlew` нельзя запустить из-за проблем с правами (например, проект был создан на ОС Windows), поэтому нужна одна дополнительная команда, чтобы это исправить. В проектах на Koltin это выглядело вот так:

```yaml
- name: Grant execute permission for gradlew
  run: chmod +x gradlew
- name: Build with Gradle
  run: ./gradlew build --info
```

Мы сделаем так же. В результате сборки (если она пройдёт успешно) как раз и появится необходимый нам файл (если быть точнее, целых два: один для отладки - debug apk, второй для релиза release apk). Нас пока будет интересовать именно debug-пакет (про release будем говорить отдельно), который мы и зальём в качестве артефакта сборки с помощью соответствующего action'а.

Ключевой нюанс: вы можете столкнуться с ошибкой вида:
```
BUILD FAILED in 42s
License for package Android SDK Build-Tools 30.0.2 accepted.
Preparing "Install Android SDK Build-Tools 30.0.2 (revision: 30.0.2)".
Warning: Failed to read or create install properties file.
##[error]Process completed with exit code 1.
```

Такое может произойти, если в вашем `build.gradle` в `buildToolsVersion` указана версия, которая ещё не доступна в конкретном worker'е ([список доступных в Ubuntu 18.04](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu1804-README.md)). Ребята из GitHub Actions, конечно же обновляют ПО, но не день в день. Поэтому при необходимости понизьте версию в своём `build.gradle` до той, что доступна в worker'е.
</details>

В чём профит? Теперь к каждому push'у у вас будет `apk`, который могут установить себе другие участники команды (например, тестировщики), и для этого не нужно скачивать репозиторий, устанавливать Android SDK и собирать всё с помощью Gradle (т.к. это уже сделал CI).

**Важно**: начиная с сегодняшней лекции, вы должны **для каждого проекта** настраивать подобным образом GitHub Actions (для этого достаточно скопировать каталог `.github` из уже настроенного проекта - вручную заново все шаги делать не нужно).

### Задача

Ваша задача на первое занятие достаточно простая: по примеру из лекции создать проект, с текстовой надписью на экране `NMedia!` вместо `Hello, World`

При этом:
* applicationId: ru.netology.nmedia
* versionName: 1.0
* minSdk (минимальная версия Android): 23 (Android 6.0)

Для проекта настроить GitHub Actions и прислать ссылку на репозиторий в личном кабинете студента.

<details>
<summary>Описание шагов выполнения</summary>

1\. Публикуете свой проект на GitHub

2\. Как и Kotlin, переходите на вкладку Actions и выбираете любой (мы всё равно заменим содержимое):

![](pic/actions.png)

3\. Заменяете содержимое на следующее (о предназначении читайте в разделе Справка выше):

```yaml
name: CI

on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]

jobs:
  build:
    runs-on: ubuntu-20.04

    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Build
        run: |
          chmod +x ./gradlew
          ./gradlew build

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v2
        with:
          name: app-debug.apk
          path: app/build/outputs/apk/debug/app-debug.apk
```

4\. Удостоверяетесь, что сборка прошла успешно и в артефактах появился `app-debug.apk`:

![](pic/build.png)

[Пример настроенного проекта](https://github.com/netology-code/and2ci).

</details>
