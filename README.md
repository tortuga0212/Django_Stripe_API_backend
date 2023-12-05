Задание:
-------

* Реализовать Django + Stripe API бэкенд со следующим функционалом и условиями:
* Django Модель `Item` с полями `(name, description, price) `
* API с двумя методами:
    * GET `/buy/{id}`, c помощью которого можно получить Stripe Session Id для оплаты выбранного Item. При выполнении
      этого метода c бэкенда с помощью python библиотеки stripe должен выполняться
      запрос` stripe.checkout.Session.create(...)` и полученный session.id выдаваться в результате запроса
    * GET `/item/{id}`, c помощью которого можно получить простейшую HTML страницу, на которой будет информация о
      выбранном `Item` и кнопка Buy. По нажатию на кнопку Buy должен происходить запрос на `/buy/{id}`, получение
      session_id и далее с помощью JS библиотеки Stripe происходить редирект на Checkout
      форму `stripe.redirectToCheckout(sessionId=session_id)`


### Установка и запуск

Установливаем Django, создаем новый проект, заходим в папку, создаем приложение, устанавливаем stripe:

```bash
pip install django
django-admin startproject stripe_site
cd stripe_site
django-admin startapp stripe_p
pip install stripe
pip freeze > requirements.txt
```
На официальном сайте stripe https://dashboard.stripe.com/test/apikeys берем Publishable key и Secret key, добавляем их в settings.py
Добавляем stripe_p.apps.StripePConfig в settings.py -> INSTALLED_APPS.
Модель продукта Product будет содержать только то, что необходимо для интеграции с Stripe.

```bash
from django.db import models


class Item(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    price = models.IntegerField(default=0) # cents

    def __str__(self):
        return self.name

    def get_str_to_dollars(self):
        return "{0:.2f}".format(self.price / 100)
```

Цена пишется в центах. Т.е. если цена=123.45$, в админ панели пишем цену 12345.

Сделаем миграции и настроим админ-панель.

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

Создаем view.py в Django для вызова Stripe API и создания сеанса проверки.
В этом файле мы вызываем stripe.checkout.Session.create и передаем некоторые параметры, которые описаны в документации Stripe.
Один из переданных параметров - метаданные metadata - словарь специальной информации, которую мы хотим предоставить этой Checkout. 
Здесь мы передаем product_id, который содержит идентификатор продукта, который мы хотим приобрести.
В конце представления мы возвращаем JsonResponse, который содержит идентификатор сеанса Checkout.
Также есть URL-адрес отмены и URL-адрес успеха, поэтому создаем две вьюшки для обработки этих URL-адресов:

```bash
class SuccessView(TemplateView):
    template_name = "success.html"


class CancelView(TemplateView):
    template_name = "cancel.html"
```
Передаем эти представления в конфигурацию корневого URL в urls.py.
Создаем папку templates с 4 файлами html:
 - для главной страницы - home.html,
 - для страницы перехода на товар - item.html,
 - для страницы успешной оплаты - success.html,
 - для страницы неуспешной оплаты - cancel.html.

Запускаем приложение:

```bash
python manage.py runserver
```
