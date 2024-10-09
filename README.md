# Создание веб-приложения на Flask для динамической обработки YAML файлов с данными о многоквартирных домах и их отображения через Apache
### Постановка задачи

**Цель:** Разработать веб-приложение на основе Python с использованием Flask, которое будет обрабатывать YAML файлы, содержащие информацию о многоквартирных домах, и отображать эти данные в виде веб-страниц через сервер Apache.

**Задачи:**

1. **Создание структуры проекта:**
   - Определить необходимую файловую структуру, включая директории для YAML файлов, HTML шаблонов и основного приложения.

2. **Разработка YAML файлов:**
   - Создать несколько YAML файлов, каждый из которых будет содержать данные о различных многоквартирных домах, включая такие параметры, как название, адрес, этажность, площадь и другие характеристики.

3. **Создание HTML шаблона:**
   - Разработать HTML шаблон, который будет использоваться для отображения информации из YAML файлов.

4. **Разработка обработчика на Flask:**
   - Написать Python скрипт, который будет обрабатывать запросы к веб-приложению, считывать соответствующий YAML файл на основе URL и генерировать HTML страницу с данными о домах.

5. **Настройка веб-сервера Apache:**
   - Настроить сервер Apache для работы с Flask приложением через модуль `mod_wsgi`, обеспечив возможность доступа к приложению по заданному домену.

6. **Тестирование и отладка:**
   - Провести тестирование приложения для проверки корректности работы с различными YAML файлами и отображения информации, а также отладить возможные ошибки.

**Ожидаемый результат:** Веб-приложение, доступное по URL, которое динамически загружает данные из YAML файлов и отображает их в структурированном виде на веб-странице.

# Решение

### Шаг 1: Создать структуру проекта

Создай структуру папок для проекта:

```
/path/to/your/app/
├── app.py
├── templates/
│   └── template.html
├── data/
│   ├── dom1.yml
│   └── dom2.yml
└── app.wsgi
```

### Шаг 2: Создать YAML файлы

Создай файлы `dom1.yml` и `dom2.yml` в папке `data`:

**data/dom1.yml**:

```yaml
template: "template.html"
houses:
  - name: "Дом 1"
    address: "Улица 1, 1"
    floors: 5
    area: 500
```

**data/dom2.yml**:

```yaml
template: "template.html"
houses:
  - name: "Дом 2"
    address: "Улица 2, 2"
    floors: 10
    area: 1000
```

### Шаг 3: Создать HTML-шаблон

Создай файл `template.html` в папке `templates`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Многоквартирные дома</title>
</head>
<body>
    <h1>Многоквартирные дома</h1>
    <div>
        {% for house in houses %}
            <h2>{{ house.name }}</h2>
            <p>Адрес: {{ house.address }}</p>
            <p>Этажность: {{ house.floors }}</p>
            <p>Площадь: {{ house.area }} м²</p>
        {% endfor %}
    </div>
</body>
</html>
```

### Шаг 4: Создать обработчик на Python

Теперь создадим файл `app.py`:

```python
from flask import Flask, render_template, abort
import yaml
import os

app = Flask(__name__)

@app.route('/<filename>.yml')
def index(filename):
    file_path = os.path.join('data', f'{filename}.yml')
    
    if not os.path.exists(file_path):
        abort(404)  # Если файл не найден, возвращаем ошибку 404
    
    with open(file_path, 'r', encoding='utf-8') as file:
        data = yaml.safe_load(file)
        return render_template(data['template'], houses=data['houses'])

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Шаг 5: Настроить Apache

Теперь настроим Apache для работы с Flask. Создай файл конфигурации `flask_app.conf`:

```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    WSGIDaemonProcess flask_app python-home=/path/to/your/venv python-path=/path/to/your/app
    WSGIScriptAlias / /path/to/your/app/app.wsgi

    <Directory /path/to/your/app>
        Require all granted
    </Directory>
</VirtualHost>
```

Замените `/path/to/your/app` на путь к вашей папке приложения.

Создай файл `app.wsgi` в директории вашего приложения:

```python
import sys
import os

sys.path.insert(0, '/path/to/your/app')

from app import app as application
```

### Шаг 6: Перезагрузить Apache

После внесения изменений перезагрузите Apache:

```bash
sudo service apache2 restart
```

### Шаг 7: Проверка работы

Теперь, если ты откроешь браузер и перейдёшь по адресу `http://yourdomain.com/dom1.yml` или `http://yourdomain.com/dom2.yml`, должен отобразиться соответствующий контент из YAML файлов.

### Ошибки и отладка

Если что-то не работает, проверь лог-файлы Apache, чтобы найти причину проблемы:

```bash
sudo tail -f /var/log/apache2/error.log
```

