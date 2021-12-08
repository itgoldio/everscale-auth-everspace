# Authorize From Everspace (rebranding from Oberton)

## Содержание
* [Сайт формирует ссылку](#website-backend-request)
    * [Пример](#website-backend-request-sample)
* [Пользователь подтверждает транзакцию](#userstory)
* [Oberton информирует сайт](#oberton)
    * [Пример](#oberton-sample)
* [Авторизация пользователя](#website-backend-response)

***

<h2 id="website-backend-request">Сайт формирует ссылку</h2>

Формирует параметры авторизации, кодирует значения ключей параметров согласно стандарта URL-encoded format https://www.w3schools.com/tags/ref_urlencode.ASP, на напримере ключа параметра callbackUrl

https://callback.url => https%3A%2F%2Fcallback.url

и отрпавляет пользователя через диплинк https://oberton.io/deeplink с этими параметрами в приложение, пример ниже:

**type**\
Константное значение **auth**\
Обязательный параметр, по нему oberton поймёт, что это запрос авторизации через кошелёк.

**id**\
Значение **string**\
Рекомендуемое значение: идентификатор запроса по которому бэкенд сайта найдёт у себя **otp**

**otp**\
Значение **string**\
Рекомендуемое значение: одноразовый пароль привязанный к генерации этой ссылки или этого QR кода. Рекомендуется связать с текущей сессией пользователя.

**callbackUrl**\
Значение **string**\
url на который кошелёк отправит POST запрос, подтверждающий владение пользователем конкретным кошельком.

**warningText**\
Значение **string**\
Сообщение которое пользователь увидит при авторизации.

<h3 id="website-backend-request-sample">Пример диплинк ссылки</h3>
https://oberton.io/deeplink?type=auth&id=testid&otp=testopt&callbackUrl=https%3A%2F%2Fcallback.url&warningText=Attention


<h2 id="userstory">Пользователь подтверждает транзакцию</h2>

Пользователь в кошельке видит запрос на авторизацию с текстом из параметра **warningText** \
Пользователь подтверждает транзакцию

<h2 id="oberton">Oberton информирует сайт</h2>

Oberton берёт адрес кошелька пользователя и otp. Подписывает получившуюся строку приватным ключём пользователя.\
Получившиюся подпись отправляет на **callbackUrl** пользователя вместе с сопутствующими данными

<h3 id="oberton-sample">Пример</h3>
ПРИЛОЖИТЬ ПРИМЕР ЗАПРОСА И BODY ОТВЕТА

<h2 id="website-backend-response">Авторизация пользователя</h2>

1. Сайт получает POST запрос от Oberton

2. По параметру id сайт находит otp у себя в локальной БД

3. Берём из ответа **addr** адрес кошелька и **pk** проверяем через БЧ что у этого кошелька есть такой публичный ключ.

4. Проверяем корректность подписи.

```javascript
   const { hash } = await this.crypto.sha256({
	data: Buffer.from(`${otp}${callbackUrl}${address}`, 'utf-8').toString(
		'base64'
	),
   });


   const { succeeded } = await this.client.crypto.nacl_sign_detached_verify({
      unsigned: Buffer.from(hash, 'hex').toString('base64'),
      signature: Buffer.from(signature.replace(/ /g, '+'), 'base64').toString('hex'),
   });
```
5. Пользователя можно авторизовать в системе.
