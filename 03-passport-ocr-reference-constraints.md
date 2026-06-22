# Passport OCR Reference and Constraints

## Назначение

Этот документ фиксирует исходные технические решения и ограничения для реализации passport OCR workflow.

Используйте его как входной reference при подготовке feature specification, technical plan, tasks и review. Не выбирайте OCR-стек заново в рамках минимальной реализации.

## 1. Целевой workflow

Система должна поддерживать backend-first flow:

```text
upload PDF/PNG
-> image normalization
-> zonal OCR
-> field extraction and normalization
-> manual review
-> confirm
-> persist confirmed record
```

Partial OCR является допустимым результатом. Если необязательное поле не распознано или распознано неоднозначно, оно остается пустым и может быть исправлено вручную перед подтверждением.

Для финального подтверждения обязательны только:

- `passportSeries`;
- `passportNumber`.

## 2. Входные данные

Минимальная реализация должна принимать ровно один файл за одну попытку обработки:

- PDF;
- PNG.

Для PDF необходимо выполнить rasterization перед OCR. Рабочий baseline для PDF rendering:

```text
Target DPI: 300
```

PNG должен поступать напрямую в тот же image normalization pipeline, который используется для изображения, полученного после rasterization PDF.

Batch upload и одновременная обработка нескольких документов не входят в минимальную реализацию.

## 3. OCR engine

Принятое решение:

```text
OCR engine: Tesseract
Recognition language: rus
Required language data: rus.traineddata
```

Требования:

- использовать Tesseract как локальный OCR engine;
- подключить `rus.traineddata`;
- не выбирать другую OCR-библиотеку в рамках минимальной реализации;
- не подключать cloud OCR в рамках минимальной реализации.

## 4. Подготовка изображения

Перед OCR изображение должно пройти preprocessing и normalization.

Обязательные части pipeline:

- grayscale;
- contrast enhancement;
- thresholding variants;
- deskew attempt;
- document boundary detection;
- perspective/crop correction;
- conservative fallback, если границы документа определены недостаточно надежно;
- zone-specific preprocessing variants.

Допустимые fallback-сценарии:

- full-page fallback, если границы документа не найдены;
- padded crop fallback, если perspective correction недостаточно надежен;
- пропуск deskew, если угол наклона нельзя определить надежно.

Минимальная реализация не должна заменять preprocessing на прямую передачу исходного файла в OCR engine.

## 5. Зонная модель

Используется zonal OCR model: распознавание выполняется по отдельным зонам нормализованного изображения.

### 5.1. Зоны и поля

`issue`:

- `issuedBy`;
- `issueDate`;
- `divisionCode`.

`identity`:

- `lastName`;
- `firstName`;
- `middleName`;
- `birthDate`;
- `birthPlace`.

`number-vertical`:

- `passportSeries`;
- `passportNumber`.

### 5.2. Baseline-координаты

Координаты задаются относительно нормализованного изображения:

```yaml
issue:
  x: 0.055
  y: 0.045
  width: 0.790
  height: 0.405
  padding: 0.015

identity:
  x: 0.300
  y: 0.515
  width: 0.625
  height: 0.420
  padding: 0.020

number-vertical:
  x: 0.885
  y: 0.120
  width: 0.095
  height: 0.780
  padding: 0.010
```

### 5.3. Правило изменения координат

Координаты и padding являются baseline-параметрами, а не неизменяемой частью системы.

Их можно корректировать, если тесты на sample documents показывают неудовлетворительное качество распознавания.

Изменения зон должны:

- выполняться отдельным явным изменением;
- сопровождаться объяснением причины;
- проходить review;
- проверяться повторным запуском OCR tests;
- не скрываться внутри unrelated implementation task.

## 6. Правила извлечения и нормализации полей

### 6.1. Серия и номер паспорта

Источник:

```text
number-vertical zone
```

Базовое правило:

- найти candidate sequence из 10 цифр;
- первые 4 цифры записать в `passportSeries`;
- следующие 6 цифр записать в `passportNumber`.

Оба поля обязательны для confirm.

### 6.2. Даты

Поля:

- `birthDate`;
- `issueDate`.

Целевой формат:

```text
dd.mm.yyyy
```

Если возможно, разделители `/` и `-` нормализуются в `.`.

### 6.3. Код подразделения

Поле:

```text
divisionCode
```

Целевой формат:

```text
ddd-ddd
```

### 6.4. Текстовые поля

Поля:

- `lastName`;
- `firstName`;
- `middleName`;
- `birthPlace`;
- `issuedBy`.

Базовая нормализация:

- trim;
- удаление шумных символов по краям;
- схлопывание повторяющихся пробелов.

### 6.5. Неоднозначные или отсутствующие значения

Если необязательное поле:

- не найдено;
- распознано неоднозначно;
- не может быть надежно нормализовано;

оно должно остаться пустым и быть доступно для ручной коррекции.

Не нужно автоматически угадывать значение или отклонять весь upload только из-за пустого необязательного поля.

## 7. Модель результата

Recognition result должен содержать:

- `passportSeries`;
- `passportNumber`;
- `lastName`;
- `firstName`;
- `middleName`;
- `birthDate`;
- `birthPlace`;
- `issueDate`;
- `divisionCode`;
- `issuedBy`.

Дополнительно recognition response может содержать:

- processing attempt identifier;
- processing status;
- raw OCR text для ручной проверки;
- минимальную metadata обработки;
- публичное сообщение об ошибке, если обработка завершилась неуспешно.

## 8. Manual review и confirm

После OCR результат должен перейти в ручную проверку.

Правила confirm:

- `passportSeries` обязателен;
- `passportNumber` обязателен;
- остальные поля допускают пустое значение;
- пользователь может скорректировать значения перед подтверждением;
- после успешного confirm сохраняется подтвержденная запись.

Автоматическое подтверждение без участия человека не входит в минимальную реализацию.

## 9. Storage policy

В минимальной реализации нужно сохранять:

- подтвержденные паспортные поля;
- минимальную metadata попытки обработки;
- статус обработки;
- публичное сообщение об ошибке для failed attempt, если применимо.

В минимальной реализации не нужно сохранять:

- исходный PDF/PNG;
- raw OCR text в БД;
- crops;
- debug images;
- preprocessing artifacts;
- OCR diagnostics как публичные artifacts.

Raw OCR text можно вернуть в recognition response для ручной проверки, но не сохранять в БД.

Временные artifacts допустимы во время обработки, если после завершения pipeline они очищаются.

## 10. Failure behavior

Невосстановимая ошибка upload, rasterization, normalization или OCR должна переводить попытку обработки в failed state.

Слабый или частичный OCR не должен автоматически переводить попытку в failed state, если система может вернуть результат для ручной проверки.

Необходимо различать:

- технический сбой pipeline;
- частично распознанный документ;
- validation error при confirm.

## 11. Out of scope

В минимальную реализацию не входят:

- выбор другой OCR-библиотеки;
- cloud OCR;
- batch processing;
- несколько типов документов в одном решении;
- production-grade OCR accuracy;
- автоматическое подтверждение по confidence score;
- обязательный frontend;
- authentication/authorization;
- постоянное хранение исходных файлов;
- постоянное хранение OCR artifacts и diagnostics.

## 12. Что можно уточнять после тестов

После запуска tests на sample documents можно пересмотреть:

- zone coordinates;
- zone padding;
- preprocessing variants;
- thresholding strategy;
- deskew behavior;
- fallback behavior;
- правила нормализации отдельных полей.

Такие изменения не должны выполняться молча. Их нужно зафиксировать как отдельное изменение к техническому plan/reference и проверить повторно.

## 13. Как использовать этот документ

При подготовке spec:

- перенесите product behavior в requirements и acceptance criteria;
- перенесите ограничения OCR engine в technical constraints;
- перенесите preprocessing и zonal model в design/technical plan;
- перенесите правила confirm в requirements и acceptance criteria;
- перенесите storage policy в persistence constraints;
- перенесите исключенные возможности в out of scope.

При постановке задачи coding agent:

```text
Прочитай OCR reference and constraints.
Не выбирай OCR-стек заново.
Не меняй baseline zones молча.
Если считаешь, что reference нужно изменить, сначала объясни причину, укажи влияние на tests и дождись review.
```
