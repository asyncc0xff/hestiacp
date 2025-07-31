# Кастомизация HestiaCP

## Описание

Данная кастомизация добавляет в HestiaCP следующие функции:

1. **Переименование доменов через веб-интерфейс** - возможность переименовать домен прямо из панели управления
2. **Автоматический выпуск Let's Encrypt SSL** - автоматическое получение SSL сертификатов при создании доменов
3. **Увеличение времени сеансов** - продление времени сеансов до 24 часов

## Установка

### 1. Backup (обязательно!)

Перед установкой обязательно создайте резервную копию:

```bash
# Создание резервной копии
cp -r /usr/local/hestia /usr/local/hestia.backup.$(date +%Y%m%d_%H%M%S)
```

### 2. Применение изменений

Скопируйте измененные файлы в соответствующие директории:

```bash
# CLI команды
cp bin/v-change-web-domain-name /usr/local/hestia/bin/
cp bin/v-add-web-domain /usr/local/hestia/bin/

# Веб-интерфейс
cp web/add/web/index.php /usr/local/hestia/web/add/web/
cp web/edit/web/index.php /usr/local/hestia/web/edit/web/
cp web/templates/pages/add_web.php /usr/local/hestia/web/templates/pages/
cp web/templates/pages/edit_web.php /usr/local/hestia/web/templates/pages/
cp web/inc/main.php /usr/local/hestia/web/inc/

# PHP конфигурация
cp src/deb/php/php.ini /usr/local/hestia/php/lib/
```

### 3. Установка прав доступа

```bash
# Установка прав на исполнение для CLI команд
chmod +x /usr/local/hestia/bin/v-change-web-domain-name
chmod +x /usr/local/hestia/bin/v-add-web-domain

# Установка прав на веб-файлы
chown www-data:www-data /usr/local/hestia/web/add/web/index.php
chown www-data:www-data /usr/local/hestia/web/edit/web/index.php
chown www-data:www-data /usr/local/hestia/web/templates/pages/add_web.php
chown www-data:www-data /usr/local/hestia/web/templates/pages/edit_web.php
chown www-data:www-data /usr/local/hestia/web/inc/main.php
```

### 4. Перезапуск сервисов

```bash
# Перезапуск веб-сервера
systemctl restart nginx
systemctl restart apache2

# Перезапуск PHP-FPM
systemctl restart php*-fpm

# Перезапуск HestiaCP
systemctl restart hestia
```

## Использование

### Переименование домена

1. Войдите в панель управления HestiaCP
2. Перейдите в раздел "Web" → "Domains"
3. Нажмите "Edit" рядом с нужным доменом
4. Измените имя домена в поле "Domain"
5. Нажмите "Save"

**Важно:** При переименовании:
- Все файлы остаются на месте
- Переименовывается только директория домена
- Автоматически перевыпускается SSL сертификат (если был Let's Encrypt)
- Включается принудительный HTTPS редирект (если был включен)

### Автоматический SSL при создании домена

1. При создании нового домена в разделе "Web" → "Add Domain"
2. По умолчанию включен чекбокс "Auto-generate Let's Encrypt SSL certificate"
3. Если чекбокс включен, то:
   - Через 30 секунд после создания домена автоматически выпускается SSL сертификат
   - Включается принудительный HTTPS редирект
   - Домен становится доступным по HTTPS

### Продленные сеансы

После установки кастомизации:
- Время сеансов увеличено до 24 часов
- Сессии не истекают при закрытии браузера
- Улучшена безопасность сессий (secure, httponly, samesite)

## Тестирование

### Тест переименования домена

```bash
# Через CLI
v-change-web-domain-name admin old-domain.com new-domain.com

# Проверка результата
ls -la /home/admin/web/
```

### Тест автоматического SSL

```bash
# Создание тестового домена
v-add-web-domain admin test-domain.com

# Проверка SSL сертификата
openssl s_client -connect test-domain.com:443 -servername test-domain.com
```

### Проверка сессий

```bash
# Проверка настроек PHP
php -i | grep session

# Проверка времени жизни сессий
grep -r "session\." /usr/local/hestia/web/
```

## Удаление кастомизации

Для удаления кастомизации восстановите файлы из резервной копии:

```bash
# Восстановление из резервной копии
cp -r /usr/local/hestia.backup.YYYYMMDD_HHMMSS/* /usr/local/hestia/

# Перезапуск сервисов
systemctl restart nginx apache2 php*-fpm hestia
```

## Совместимость

- **HestiaCP версия:** 1.6.x и выше
- **Операционные системы:** Ubuntu 20.04+, Debian 11+
- **Веб-серверы:** Nginx, Apache
- **PHP версии:** 7.4+

## Безопасность

- Все изменения сохраняют существующие механизмы безопасности
- Добавлены дополнительные параметры безопасности для сессий
- Автоматический SSL использует стандартные механизмы Let's Encrypt

## Поддержка

При возникновении проблем:

1. Проверьте логи: `tail -f /var/log/hestia/error.log`
2. Проверьте права доступа к файлам
3. Убедитесь, что все сервисы перезапущены
4. Восстановите из резервной копии при необходимости

## Изменения в файлах

### Добавленные функции:

1. **bin/v-change-web-domain-name** - автоматический перевыпуск SSL при переименовании
2. **bin/v-add-web-domain** - автоматический SSL при создании домена
3. **web/add/web/index.php** - обработка чекбокса автоматического SSL
4. **web/edit/web/index.php** - обработка переименования домена
5. **web/templates/pages/add_web.php** - чекбокс автоматического SSL
6. **web/templates/pages/edit_web.php** - активация поля домена для редактирования
7. **web/inc/main.php** - настройки продленных сессий
8. **src/deb/php/php.ini** - конфигурация PHP сессий

### Общий объем изменений: ~70 строк кода 