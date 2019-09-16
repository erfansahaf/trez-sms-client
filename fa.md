# کتابخانه ارسال اس‌ام‌اس ترز

این کتابخانه برای استفاده از وب سرویس اس‌ام‌اس ترز در جاوا اسکریپت (Node.js) طراحی و پیاده سازی شده است. در بخش زیر نحوه ی استفاده از آن را خواهید دید.

## نصب و تنظیمات

جهت نصب این کتابخانه میتوانید از طریق NPM اقدام کنید:
```bash
npm install trez-sms-client --save
```
با اجرای دستور فوق، این پکیج به لیست دیپندنسی های پروژه شما افزوده خواهد شد. حال میتوانید یک آبجکت از کلاس TrezSmsClient ایجاد کنید:
```js
const TrezSmsClient = require("trez-sms-client");
const client = new TrezSmsClient("username", "password");
```
سازنده ی کلاس TrezSmsClient دو پارامتر نام کاربری و رمز عبور حساب پنل اس‌ام‌اس شما را دریافت میکند.

## احراز هویت

با استفاده از متد های زیر میتوانید یک کد فعال سازی جهت احراز هویت یک شماره موبایل ارسال نمایید.
این کد بعدا میتواند توسط خود سیستم اعتبارسنجی شود و صحت آن مورد بررسی قرار گیرد.

### ارسال کد بصورت اتوماتیک

با استفاده از متد زیر میتوانید بصورت اتوماتیک یک کد تصادفی برای کاربر با متن دلخواه در انتهای اس ام اس ارسال کنید.
هنگام موفقیت آمیز بودن عملیات میتوانید از کد برگشتی برای بررسی وضعیت پیام ارسال شده در متد مربوطه استفاده نمایید.

```js
client.autoSendCode("09301234567", "Signiture Footer For Branding")
    .then((messageId) => {
        console.log("Sent Message ID: " + messageId);
    })
    .catch(error => console.log(error));
```

### بررسی صحت کد ارسال شده

پس از ارسال کد فعالسازی از طریق متد قبلی، از این متد جهت اعتبارسنجی کد کاربر میتوانید استفاده نمایید:

```js
client.checkCode("09301234567", "595783")
    .then((isValid) => {
        if (isValid) {
            console.log("Code 595783 for this number 09301234567 is valid and verified.");
        }
        else {
            console.log("Provided code for that number is not valid!");
        }
    })
    .catch(error => console.log(error));
```

### ارسال کد دلخواه

از طریق این روش میتوانید کد مورد نظر را به همراه متن دلخواه به کاربر ارسال کنید.
.این روش مانند متد ارسال پیام کار میکند

هنگام موفقیت آمیز بودن عملیات میتوانید از کد برگشتی برای بررسی وضعیت پیام ارسال شده در متد مربوطه استفاده نمایید.

```js
client.manualSendCode("09301234567", "Verification Code: 595783 \nTrez WebService SMS")
    .then((messageId) => {
        console.log("Sent Message ID: " + messageId);
    })
    .catch(error => console.log(error));
```

## ارسال پیام

برای ارسال پیام میتوانید از متد `sendMessage` استفاده کنید:
```js
client.sendMessage(sender, numbers, message, groupId)
	.then((receipt) => {
		console.log("Receipt: " + receipt);
	})
	.catch((error) => {
		// If there is an error, we'll catch that
		console.log(error.isHttpException, error.code, error.message);
	});
```
| Arg | Type | Example | Description |
|--|--|--|--|
| sender | string | "5000248889" | Your number for sending messages |
| numbers | string\|array | ["0912xxxxxxx", "0919xxxxxxx"] | List of numbers you wish to receive the Message
| message | string | "Hello Guys!" | The message you want to send |
| groupId | integer | "147852369" | An unique ID for tracking the sent messages |

از این متد برای ارسال تکی نیز میتوانید استفاده کنید.


## ارسال پیام دسته ای یا متناظر

از این متد برای ارسال پیام نظیر به نظیر استفاده میشود و هر پیام به شماره متناظر خود ارسال خواهد شد. با استفاده از این قابلیت میتوانید پیام متفاوتی را برای هر شماره بصورت اختصاصی تنها با یک بار فراخوانی ارسال کتید.

```js
const recipients = [
							  // Unique ID,  Mobile Number, Message
	client.createRecipientObject("11445599", "0912xxxxxxx", "This is a message"),
	client.createRecipientObject("99854710", "0930xxxxxxx", "This is another message"),
];

client.sendBatchMessage(sender, recipients, groupId)
	.then((result) => {
		console.log("Result: " + result); // [{"Id":11445599,"Mobile":"09123456789","Result":2000}, {"Id":99854710,"Mobile":"09301234567","Result":2001}]
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```

| Arg | Type | Example | Description |
|--|--|--|--|
| sender | string | "5000248889" | Your number for sending messages |
| recipients | array | [client.createRecipientObject("ID1", "Number", "Message")] | An array consisted of recipient objects
| groupId | integer | "147852369" | An unique ID for tracking the sent messages |

## وضعیت پیام

از این متند برای مشاهده وضعیت پیام های ارسالی در مراحل قبل میتوان استفاده کرد.

```js
client.messageStatus(groupId)
	.then((result) => {
		console.log(result.received, result.status, result.message, result.number);
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```

| Arg | Type | Example | Description |
|--|--|--|--|
| groupId | integer | "147852369" | The groupId which you specified in sending message stage |

## وضعیت پیام دسته ای یا متناظر

از این متند برای مشاهده وضعیت پیام هایی که در مراحل قبل بصورت دسته ای یا نظیر به نظیر ارسال شده اند میتوان استفاده کرد.

```js
client.batchMessageStatus(ids)
	.then((result) => {
		console.log(result);
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```

| Arg | Type | Example | Description |
|--|--|--|--|
| ids | array | ["11445599", "99854710"] | The ids that you used to make recipients objects |

## پیام های دریافتی

با استفاده از این متد میتوانید به پیام های دریافتی خط مورد نظر دسترسی داشته باشید:

```js
const from = 1557695398; // from date in timestamp
const to = 1557868230; // till date in timestamp
const page = 1;

client.receivedMessages(receiver, from, to, page)
	.then((result) => {
		console.log(result.totalPages, result.currentPage, result.messages); // 10, 1, [{from: 09381234567, date: 1557695511, message: 'سلام'}]
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```

| Arg | Type | Example | Description |
|--|--|--|--|
| receiver | string | "5000248889" | Your receiving number |
| from | timestamp | 1557695398 | Start Date (from) |
| to | timestamp | 1557868230 | End Date (from) |
| page | integer | 1 | Number of the target page in pagination |


## موجودی حساب

از طریق این متد میتوانید موجودی حال حاضر حساب خود را به ریال محاسبه کنید:

```js
client.accountCredit()
	.then((credit) => {
		console.log(credit + " Rials"); // 500000 Rials
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```

## قیمت ها

جهت دریافت تعرفه ارسال پیامک از این متد استفاده کنید

```js
client.prices()
	.then((prices) => {
		console.log("Farsi: " + prices.fa + " Rials, English: " + prices.en + " Rials"); // Farsi: 129 Rials, English: 295 Rials
	})
	.catch((error) => {
		console.log(error.isHttpException, error.code, error.message);
	});
```
