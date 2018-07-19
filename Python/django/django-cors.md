```python
  pip install django-cors-headers

  setting.py
  INSTALLED_APPS = (
    ...
    'corsheaders',
    ...
  )

  MIDDLEWARE = [  # Or MIDDLEWARE_CLASSES on Django < 1.10
    ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
    ]
  CORS_ORIGIN_WHITELIST = (
    'google.com',
    'hostname.example.com',
    'localhost:8000',
    '127.0.0.1:9000'
    )
  ```
