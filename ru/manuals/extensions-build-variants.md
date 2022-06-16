---
layout: manual
language: ru
github: https://github.com/defold/doc
title: Нативные расширения - Варианты сборки
brief: Это руководство описывает различные варианты сборок, которые может создавать Defold, и то, как они взаимодействуют с нативными расширениями и движком.
---

# Нативные расширения - Варианты сборки

## Варианты Сборки

Когда вы собираете игру, вам нужно выбрать тип движка, который вы хотите использовать.

  * Debug
  * Release
  * Headless

Эти различные версии также называются `вариантами сборки`.

Примечание: Когда вы выберете `Build and run`, вы получите отладочную версию.

### Debug

В этом типе исполняемого файла еще остались функции отладки, такие как профилирование, протоколирование и горячая перезагрузка. Этот вариант выбирается во время разработки игры.

### Release

В этом варианте функции отладки отключены. Этот вариант выбирается, когда игра готова к выпуску в магазин приложений.

### Headless

Этот исполняемый файл запускается без графики и звука. Это означает, что вы можете запускать игровые юнит/дымовые тесты на CI-сервере или даже иметь его в качестве игрового сервера в облаке.

## App Manifest

С помощью функции Native Extensions вы можете не только добавлять в движок нативный код, но и удалять стандартные части движка. Например, если вам не нужен физический движок, вы можете удалить его из исполняемого файла.

Мы поддерживаем это с помощью файла, называемого `App Manifest` (.appmanifest). В таком файле вы можете указать, какие библиотеки или символы удалить, или, возможно, добавить флаги компиляции

Эта функция находится в стадии разработки и совершенствования.

### Комбинированный контекст

Манифест приложения фактически имеет ту же структуру и синтаксис, что и манифест расширения. Это позволяет нам объединять контексты для одной платформы вместе при окончательной сборке.

А сам Defold имеет свой собственный манифест сборки в качестве основы (`build.yml`). Для каждого расширения, которое собирается, манифесты объединяются следующим образом:

	manifest = merge(game.appmanifest, ext.manifest, build.yml)

Это делается для того, чтобы пользователь мог переопределить поведение по умолчанию движка, а также каждого расширения. И, на последнем этапе соединения, мы объединяем манифест приложения с манифестом Defold:

	manifest = merge(game.appmanifest, build.yml)

### Редактирование

В настоящее время этот процесс можно выполнить вручную, но мы рекомендуем нашим пользователям использовать [Manifestation](https://britzl.github.io/manifestation/) инструмент для создания манифеста приложения. В конечном итоге создание и изменение манифестов приложений будет осуществляться в редакторе.

### Синтаксис

Вот пример из [Manifestation](https://britzl.github.io/manifestation/) для справки (Может быть изменен. Не копируйте этот файл напрямую. Вместо этого используйте онлайн-инструмент):

	platforms:
	    x86_64-osx:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    x86_64-linux:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    js-web:
	        context:
	            excludeLibs: []
	            excludeJsLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    wasm-web:
	        context:
	            excludeLibs: []
	            excludeJsLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    x86-win32:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    x86_64-win32:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    armv7-android:
	        context:
	            excludeLibs: []
	            excludeJars: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    armv7-ios:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []
	    arm64-ios:
	        context:
	            excludeLibs: []
	            excludeSymbols: []
	            libs: []
	            linkFlags: []


#### Белый список

Для всех ключевых слов мы применяем фильтр белых списков. Это делается для того, чтобы избежать незаконной работы с путями и доступа к файлам вне папки загрузки сборки.

#### linkFlags

Здесь вы можете добавить флаги для компилятора конкретной платформы

#### libs

Этот флаг используется только в том случае, если вы хотите добавить библиотеку, которая является частью платформы или Defold SDK. Все библиотеки в расширениях вашего приложения добавляются автоматически, и вы не должны добавлять их в этот флаг. Вот пример, в котором 3D-физика удалена из движка:

    x86_64-linux:
        context:
            excludeLibs: ["physics","LinearMath","BulletDynamics","BulletCollision"]
            excludeSymbols: []
            libs: ["physics_2d"]
            linkFlags: []

#### Флаги исключений

Эти флаги используются для удаления вещей, ранее определенных в контексте платформы. Вот пример того, как удалить расширение Facebook из движка (обратите внимание на `(.*)`, которое является регулярным выражением, чтобы помочь удалить нужные элементы).

    armv7-android:
        context:
            excludeLibs: ["facebookext"]
            excludeJars: ["(.*)/facebooksdk.jar","(.*)/facebook_android.jar"]
            excludeSymbols: ["FacebookExt"]
            libs: []
            linkFlags: []

#### Где список всех флагов, библиотек, символов???

Мы могли бы разместить некоторые из них здесь, но мы также считаем, что наше время лучше потратить на завершение функции перемещения конфигурации манифеста в редактор, сделав это простым шагом для пользователя.

Тем временем мы будем обновлять инструмент [Manifestation](https://britzl.github.io/manifestation/).