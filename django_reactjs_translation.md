 Let's walk through an example of how to set up translations in a Django REST framework (DRF) backend and a React frontend. This will involve marking strings for translation in Django, creating translation files, and setting up i18n in React.

Step-by-Step Guide
1. Setting Up Django for Translations
Step 1.1: Mark Strings for Translation

In your Django views, serializers, or models, use gettext (or its alias _) to mark strings for translation.

views.py:

``` python

from django.http import JsonResponse
from django.utils.translation import gettext as _

def my_view(request):
    message = _("Welcome to my site.")
    return JsonResponse({'message': message})
serializers.py:

python

from rest_framework import serializers
from django.utils.translation import gettext_lazy as _

class MySerializer(serializers.Serializer):
    message = serializers.CharField(default=_("Default message"))
```
Step 1.2: Create Translation Files

Run the makemessages command to create translation files for the languages you want to support.

``` bash
django-admin makemessages -l es
```
This will create a file django.po in the locale/es/LC_MESSAGES directory for Spanish translations.

Step 1.3: Edit Translation Files

Open the django.po file and add translations.

locale/es/LC_MESSAGES/django.po:

``` plaintext
msgid "Welcome to my site."
msgstr "Bienvenido a mi sitio."
```
Step 1.4: Compile Translation Files

Compile the message files to binary format.

``` bash

django-admin compilemessages
```
Step 1.5: Middleware to Activate Translation

You might want to create middleware to activate the user's language choice.

middleware.py:

``` python

from django.utils import translation

class SetLanguageMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        language = request.GET.get('lang', 'en')
        translation.activate(language)
        response = self.get_response(request)
        translation.deactivate()
        return response
```
Add this middleware to your MIDDLEWARE settings.

```python

MIDDLEWARE = [
    # other middleware
    'path.to.SetLanguageMiddleware',
]
```
Step 1.6: REST API Endpoint

Create an endpoint to get translated messages.

views.py:

``` python

from rest_framework.views import APIView
from rest_framework.response import Response
from django.utils.translation import gettext as _

class MyView(APIView):
    def get(self, request):
        message = _("Welcome to my site.")
        return Response({'message': message})
```
urls.py:

```python

from django.urls import path
from .views import MyView

urlpatterns = [
    path('api/message/', MyView.as_view(), name='message'),
]
```
2. Setting Up React for Translations
Step 2.1: Install i18next and React i18next

In your React project, install i18next and react-i18next.

```bash

npm install i18next react-i18next i18next-http-backend i18next-browser-languagedetector
```
Step 2.2: Initialize i18next

Create an i18n.js file to initialize i18next.

src/i18n.js:

```javascript

import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import HttpBackend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(HttpBackend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    debug: true,
    interpolation: {
      escapeValue: false,
    },
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
  });

export default i18n;
```
Step 2.3: Create Translation JSON Files

Create translation JSON files for each language.

public/locales/en/translation.json:

```json

{
  "Welcome to my site.": "Welcome to my site."
}
```
public/locales/es/translation.json:

```json

{
  "Welcome to my site.": "Bienvenido a mi sitio."
}
```
Step 2.4: Use Translations in React Components

Use the useTranslation hook to access translations in your components.

src/App.js:

```javascript

import React, { useEffect, useState } from 'react';
import { useTranslation } from 'react-i18next';
import i18n from './i18n';

function App() {
  const { t } = useTranslation();
  const [message, setMessage] = useState('');

  useEffect(() => {
    fetch('/api/message/?lang=' + i18n.language)
      .then(response => response.json())
      .then(data => setMessage(data.message));
  }, [i18n.language]);

  return (
    <div>
      <h1>{t('Welcome to my site.')}</h1>
      <p>{message}</p>
      <button onClick={() => i18n.changeLanguage('es')}>ES</button>
      <button onClick={() => i18n.changeLanguage('en')}>EN</button>
    </div>
  );
}

export default App;
```
Step 2.5: Provide Translation Context

Wrap your app with the I18nextProvider.

src/index.js:

``` javascript

import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { I18nextProvider } from 'react-i18next';
import i18n from './i18n';

ReactDOM.render(
  <React.StrictMode>
    <I18nextProvider i18n={i18n}>
      <App />
    </I18nextProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```
Summary
- Backend (Django):

Mark strings for translation using gettext.
Create and compile translation files.
Activate translations based on user preference.
- Frontend (React):

Initialize i18next for managing translations.
Create JSON translation files.
Use the useTranslation hook to access and display translated strings.
Fetch translated messages from the Django API and update the UI accordingly.
By following these steps, you can set up a multilingual application with Django and React, allowing users to switch languages seamlessly.