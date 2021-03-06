# Задание 3

Мобилизация.Гифки – сервис для поиска гифок в перерывах между занятиями.

Сервис написан с использованием [bem-components](https://ru.bem.info/platform/libs/bem-components/5.0.0/).

Работа избранного в оффлайне реализована с помощью технологии [Service Worker](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API/Using_Service_Workers).  

Для поиска изображений используется [API сервиса Giphy](https://github.com/Giphy/GiphyAPI).  

---
1. Проблема «Более надежное кэширование на этапе `fetch`».  
Решить данную проблему можно двумя способами: либо добавить файл `gifs.html` в кэш на этапе установки Service Worker'a (`install`), либо добавить его в функцию `needStoreForOffline()`.  
Исходя из требования дополнительного задания я решил использовать первый споособ. Ведь при активации Service Worker'a (`activate`) вызывается метод `clients.claim()`, и в совокупности такое решение данной проблемы позволяет переключиться в офлайн-режим после первого запроса.
1. Проблема «Невозможно обновить статику из директорий `vendor` и `assets`».  
Решение данной проблемы заключается в изменении версии в константной переменной `CACHE_VERSION` на иное, тем самым при активации "нового" Service Worker'a (`activate`) будет вызвана функция `deleteObsoleteCaches()`.  
1. Проблема «Разложить файлы красиво».  
Для решения данной проблемы достаточно перенести файл `service-worker.js` в корневую директорию проекта (в данном случае `entrance-task-3`), а также изменить путь к `service-worker.js` в файле `blocks.js`. Требуется это для того, чтобы Service Worker мог обрабатывать все запросы данного сайта. Связано это с тем, что область видимости Service Worker (`scope`) ограничивается директорией (и поддиректориями), в которой он находится.
1. При добавлении гифки в избранное при отключенном интернет-соединении (например, оно оборвалось), она добавляется в хранилище `KvKeeper`, но не добавляется в кэш.  
Для решения данной проблемы был добавлен массив `brokenFarovites`, в который записываются все "битые элементы", и при следующем запросе с интернет-соединением добавляет эти элементы в кэш.
1. При удаления гифки из избранного, она не удаляется из кэша.  
Для решения данной проблемы был добавлен обработчик события `favorite:remove`.
1. Нужно ли при скачивании сохранять ресурс для оффлайна?  
```js
function needStoreForOffline(cacheKey) {
	return cacheKey.includes('vendor/') ||
		cacheKey.includes('assets/') ||
		cacheKey.endsWith('jquery.min.js');
}
```
Не нужно, так как все файлы были закэшированы на этапе установки Service Worker'a.
---

* Вопрос №1: зачем нужен этот вызов? `skipWaiting()`  
Этот вызов необходим при обновлении Service Worker'a. После регистрации обновленный Service Worker будет находиться в т.н. режиме ожидания, когда он не обрабатывает запросы. Этим занимается старая версия Service Worker. Вызов этого метода позволяет обновленному Service Worker начать обрабатывать запросы.
* Вопрос №2: зачем нужен этот вызов? `clients.claim()`  
После регистрации Service Worker нужно еще активировать. По умолчанию это происходит после перезагрузки страницы. Вызов этого метода немедленно активирует Service Worker.
* Вопрос №3: для всех ли случаев подойдет такое построение ключа? `url.origin + url.pathname`  
Нет, если в запросе будет присутствовать параметр `url.search`.
* Вопрос №4: зачем нужна эта цепочка вызовов?  
```js
return Promise.all(
	names.filter(name => name !== CACHE_VERSION)
		.map(name => {
			console.log('[ServiceWorker] Deleting obsolete cache:', name);
			return caches.delete(name);
		})
);
```
Чтобы удалить старую версию кэша, не повреждая данные.
* Вопрос №5: для чего нужно клонирование?  
С полученным `response` нам надо сделать две вещи: положить в кэш и ответить на событие (т.е. вернуть его).
Так как объект `response` может быть использован только один раз, клонирование позволяет нам создать копию для нужд кэша.
