+++
date = '2025-03-02'
title = 'Сборка go-приложений на fyne под Android'
+++

---
## Содержание:

- [Prerequisites](#prerequisites)
- [Установка Android Sdk & Ndk](#установка-android-sdk--ndk)
- [Создание go-приложения](#создание-go-приложения)
- [Готовим телефон к работе](#готовим-телефон-к-работе)
- [Запуск приложения на телефоне](#запуск-приложения-на-телефоне)
- [Makefile для удобства работы](#makefile-для-удобства-работы)

---

## Prerequisites:

Сперва установим всё необходимое:
- [go](https://go.dev/dl/)
- [fyne](https://docs.fyne.io/), а также `go install fyne.io/fyne/v2/cmd/fyne@latest`
- на случай если вы сейчас на `Windows`, то нужен еще и [gcc for windows](http://www.equation.com/servlet/equation.cmd?fa=fortran). Он понадобиться и для обычной компиляции под сам `Windows`. Не забудьте добавить путь к нему в `PATH`.

---

## Установка Android Sdk & Ndk:

Отдельно поговорим про установку Android `Sdk` и `Ndk`. 
Сперва создадим место под них в `~`:

```bash
mkdir Android
```

Затем по [этой ссылке](https://github.com/android/ndk/wiki/Unsupported-Downloads) скачиваем `Ndk` версии `r24` под нужную вам ОС. Распаковываем `Ndk` в нашу директорию `Android`:

```bash
sudo apt install unzip
cd Загрузки
unzip anroid-ndk-r24-linux.zip -d ~/Android
cd ~/Anrdoid
mv ndk-r24-linux ndk-bundle
```

Чтобы установить `Sdk` переходим по [этой ссылке](https://developer.android.com/tools/releases/platform-tools?hl=ru) и выбираем нужную ОС. Нужно будет согласится с условиями пользования. Теперь устанавливаем:

```bash
cd Загрузки
unzip platform-tools-latest-linux.zip -d ~/Android
cd ~/Android
mv platform-tools Sdk
```

Мы установили всё необходимое, теперь нужно настроить переменные окружения. Вот как это можно сделать в `Linux`:

```bash
emacs ~/.bashrc
```

Вместо `emacs` можете использовать свой любимый редактор (но я буду вас осуждать за это). И добавляем в конец файла следующий фрагмент в котором заменяем все вхождения {$user} на свое имя пользователя:

```bash
export ANDROID_HOME="/home/{$user}/Android"
export ANDROID_NDK_HOME="/home/{$user}/Android/ndk-bundle"
export ANDROID_SDK_HOME="/home/{$user}/Android/Sdk"
export PATH=$PATH:$ANDROID_HOME:$ANDROID_NDK_HOME:$ANDROID_SDK_HOME
```

Теперь проверим что мы всё сделали правильно:

```bash
source ~/.bashrc
which adb # должно вывести: /home/{$user}/Android/Sdk/adb
```

---

## Создание go-приложения:

Теперь можно приступить к созданию самого приложения:

```bash
mkdir my-app && cd my-app
emacs main.go
```

И добавляем следующий код:

```go
package main

import (
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("my-app")
	l := widget.NewLabel("hello")
	w.SetContent(l)
	w.ShowAndRun()
}
```

Возвращаемся в терминал и проверяем что код работает:

```bash
go mod tidy
go run main.go
```

---

## Готовим телефон к работе:

Перво-наперво нужно настроить устройство на котором будет запускаться приложение. Есть несколько разных вариантов, но я остановлюсь на работе через `USB`. Способы для разных версий `Android` могут отличаться, в моем случае нужно было сделать следующее: заходим в настройках в `О телефоне`, много раз нажимаем на `Версия MIUI` пока не появится сообщение о получении прав разработчика, далее переходим в `Расширенные настройки/Для разработчиков` где включаем `Режим разработчика`, после чего ищем раздел `Отладка` и включаем опцию `Отладка по USB`. Когда устройство будет подготовлено подключаем его по `USB` кабелю к вашей машине.

---

## Запуск приложения на телефоне:

Прежде чем запустить ваше приложение на телефоне необходимо сделать иконку этого приложения. Оно должно быть разрешением `100х100` пикселей и в формате `png`. Я обычно называю его `logo.png`. 

Теперь у нас есть всё необходимое для запуска. Введите в терминале в корне вашего проекта следующее:

```bash
go mod tidy
fyne package -os android -appID my.app -icon logo.png -name myapp
adb -d install -r myapp.apk
```

Возможно для пользования `adb` придется еще немного повозиться, но я сейчас не могу точно сказать что именно нужно будет сделать. Как это принято говорить: оставим читателю в качестве упражнения)

---

## Makefile для удобства работы:

Создадим следующий `Makefile`:

```make
build:
	go mod tidy
	fyne package -os android -appID my.app -icon logo.png -name myapp

debug:
	adb -d install -r myapp.apk

install: build debug
```

Теперь после внесения изменений в код приложения всё что нужно будет сделать для их проверки на телефоне это вызвать следующую команду:

```bash
make install
```

---

Поздравляю! Теперь у вас есть свое Android-приложение написанное на `Go`!

---

[наверх](#содержание)