# Пример использования AvaCapoTalkingHead

В этом руководстве объясняется, как использовать класс `AvaCapoTalkingHead` для создания 3D-анимированного аватара, который может говорить и синхронизировать движения губ с аудио. Генерация речи и визем выполняется на стороне бэкенда.

## Необходимые зависимости

Убедитесь, что в проекте подключена необходимая библиотека:

```javascript
import { AvaCapoTalkingHead } from "./modules/avaCapoSdk.mjs";
```

Для работы этой библиотеки необходимо также указать importmap:

```javascript
<script type="importmap">
        {
          "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.161.0/build/three.module.js/+esm",
            "three/examples/": "https://cdn.jsdelivr.net/npm/three@0.161.0/examples/",
            "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.161.0/examples/jsm/"
          }
        }
    </script>
```

##  1. Создание экземпляра класса `AvaCapoTalkingHead`

Необходимо создать экземпляр класса AvaCapoTalkingHead, указав HTML-элемент для отображения аватара и передав набор параметров конфигурации.

```javascript
const nodeAvatar = document.getElementById("avatar");

head = new AvaCapoTalkingHead(nodeAvatar, {
    cameraView: "head" // Установить вид камеры "head" для портретного режима
});
window.head = head;
```

##  2. Загрузка и отображение аватара

Для отображения аватара необходимо использовать метод showAvatar, указав URL модели:

```javascript
await head.showAvatar({
        url: "./avatars/brunette.glb"
});
```

##  3. Разблокировка Web Audio API

Если Web Audio API заблокирован до взаимодействия с пользователем, его можно разблокировать следующим образом:

```javascript
function headLoaded() {
  if (head.audioCtx.state === "suspended") {
    if ("ontouchstart" in window) {
      let unlockWebAudioAPI = function () {
        head.audioCtx.resume().then(() => {
          document.body.removeEventListener("touchstart", unlockWebAudioAPI);
          document.body.removeEventListener("touchend", unlockWebAudioAPI);
        });
      };
      document.body.addEventListener("touchstart", unlockWebAudioAPI, false);
      document.body.addEventListener("touchend", unlockWebAudioAPI, false);
    }
  }
}
```

##  4. Генерация речи

Для генерации речи как пример используется функция ttsLoadMP3, которая отправляет текст на сервер, получает MP3-файл и данные о виземах и таймингах. Пример функции:

```javascript
async function ttsLoadMP3(text) {
  try {
    const speakButton = document.querySelector("#speakButton");

    // Отправка текста для генерации аудио
    let response = await fetch(`${ttsProxyGenerateEndpoint}?text=${encodeURIComponent(text)}`, {
      method: "GET"
    });

    if (response.ok) {
      const audioData = await response.blob(); // Получение аудио в формате blob

      // Отправка запроса для получения таймингов и виземов
      response = await fetch(ttsProxyProcessEndpoint, {
        method: "POST",
        headers: {
          'Content-Type': 'application/json'
        }
      });

      if (response.ok) {
        const json = await response.json();

        // Преобразование аудиоданных в ArrayBuffer для дальнейшей обработки
        const arraybuffer = await audioData.arrayBuffer();
        
        return {
          json,
          arraybuffer,
        };
      } else {
        console.log('Ошибка обработки аудио:', response.status, response.statusText);
      }
    } else {
      console.log('Ошибка генерации MP3:', response.status, response.statusText);
    }
  } catch (error) {
    console.log(error);
  }

  return null; // Возврат null в случае ошибки
}

```
## 5. Воспроизведение речи и анимации

```javascript
speakButton.addEventListener("click", () => {
  const text = textInput.value.trim();

  if (text && !speakButton.classList.contains("disabled")) {
    ttsLoadMP3(text).then((result) => {
      if (result) {
        head.speakResponses(result.json, result.arraybuffer);  // Воспроизведение речи и анимации
      }
    });
  }
});
```

## Вывод

Класс AvaCapoTalkingHead позволяет создавать анимированного 3D-аватара с возможностью синхронизации речи, движений губ и жестов. Генерация речи и данных для анимации выполняется на стороне бэкенда, а затем передается аватару для воспроизведения на клиентской стороне.

