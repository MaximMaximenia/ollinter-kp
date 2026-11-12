# ТЗ: подключение Meta Pixel и событий воронки

Сайт: ollinter.com
ID пикселя: 2420908798717322

## 1. Базовый код

Вставить в `<head>` всех страниц сайта, включая все языковые версии (/de/, /en/, /ru/).

```html
<!-- Meta Pixel Code -->
<script>
!function(f,b,e,v,n,t,s)
{if(f.fbq)return;n=f.fbq=function(){n.callMethod?
n.callMethod.apply(n,arguments):n.queue.push(arguments)};
if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
n.queue=[];t=b.createElement(e);t.async=!0;
t.src=v;s=b.getElementsByTagName(e)[0];
s.parentNode.insertBefore(t,s)}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', '2420908798717322');
fbq('track', 'PageView');
</script>
<noscript><img height="1" width="1" style="display:none"
src="https://www.facebook.com/tr?id=2420908798717322&ev=PageView&noscript=1"
/></noscript>
<!-- End Meta Pixel Code -->
```

**Важно про SPA.** Сайт отрисовывает страницы без перезагрузки, поэтому `PageView` из базового кода сработает только один раз — при первом открытии. На каждую смену маршрута надо дополнительно вызывать:

```js
fbq('track', 'PageView');
```

Один вызов на один переход, без дублей.

## 2. События

Шесть событий. Вызывать ровно в описанных местах, имена и ключи параметров не менять.

### 2.1 Клик по главной кнопке

Кнопка «Lehrkraft auswählen» — и в первом экране, и в футере.

```js
fbq('trackCustom', 'TeachersClick', {
  position: 'hero',        // 'hero' или 'footer'
  language: 'de'           // язык интерфейса
});
```

### 2.2 Открытие карточки преподавателя

```js
fbq('track', 'ViewContent', {
  content_ids: [teacher_id],
  content_type: 'product',
  content_name: subject,   // 'english', 'german' и т.д.
  value: price,            // число, цена часа
  currency: 'EUR'
});
```

### 2.3 Начало бронирования

Нажата кнопка бронирования слота — **до** формы регистрации.

```js
fbq('track', 'InitiateCheckout', {
  content_ids: [teacher_id],
  value: price,
  currency: 'EUR'
});
```

### 2.4 Регистрация

Только после успешного создания аккаунта, **не** по клику на кнопку.

```js
fbq('track', 'CompleteRegistration', {
  method: 'email'          // 'email' или 'google'
});
```

### 2.5 Клик по кнопке оплаты

До подтверждения платежа, в момент перехода к оплате.

```js
fbq('track', 'AddPaymentInfo', {
  content_ids: [teacher_id],
  value: price,
  currency: 'EUR'
});
```

### 2.6 Оплата подтверждена

Только после фактического подтверждения платежа платёжной системой.

```js
fbq('track', 'Purchase', {
  content_ids: [teacher_id],
  value: price,
  currency: 'EUR'
});
```

## 3. Требования

- **Имена событий и параметров менять нельзя** — на них завязана оптимизация рекламы.
- `value` передавать **числом**, без символа валюты и без кавычек. `29`, а не `"29 €"`.
- `currency` всегда строкой `'EUR'`.
- Каждое событие срабатывает **один раз** за действие. Дубли ломают статистику.
- `Purchase` отправлять только при реальном подтверждении оплаты. Не по редиректу на страницу «спасибо», если на неё можно зайти напрямую.
- **Не вырезать параметры из URL.** Параметры `fbclid` и все `utm_*` должны доживать до посадочной страницы и сохраняться при внутренних переходах. На них держится вся связка «объявление → действие на сайте».
- Если сайт сам ставит редиректы (например, с `/for/languages` на `/de/for/languages`), параметры запроса должны переноситься в адрес назначения.

## 4. Проверка

Поставить расширение **Meta Pixel Helper** для Chrome и пройти путь целиком: главная → клик по кнопке → карточка преподавателя → бронирование → регистрация → оплата.

Убедиться, что:
- каждое событие появилось в списке один раз;
- параметры пришли заполненными, а не пустыми;
- нет ошибок и предупреждений красным.

Скриншот из Pixel Helper после прохода — в подтверждение, что всё работает.
