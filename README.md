# Project_template

### Задание 1. Анализ и планирование

### 1. Описание функциональности монолитного приложения

**Управление отоплением:**

Пользователи могут 
- Изменять вручную значение переменной, отвечающей за измеряемую датчиком температуру (через эндпойнт PATCH /api/v1/sensors/:id/value);  
- Изменять статус (вкл/выкл) датчика (через эндпойнт PUT /api/v1/sensors/:id).  
Система не поддерживает функцию управления (даже, если управляемый датчик имеет исполнительный механизм, например, это термостат, структура данных, описывающая датчики, не имеет параметра, управляющего его уставкой; value – измеряемая температура и его ручное изменение пользователем у реального устройства не приведет к изменению поведения исполнительного устройства системы отопления);  
```
// Sensor represents a smart home sensor type Sensor struct {  
ID int json:"id"  
Name string json:"name"  
Type SensorType json:"type"  
Location string json:"location"  
Value float64 json:"value"  
Unit string json:"unit"  
Status string json:"status"  
LastUpdated time.Time json:"last_updated"  
CreatedAt time.Time json:"created_at"  
}  
```
**Мониторинг температуры:**

Пользователи могут получить 
- Список всех датчиков (через эндпойнт GET /api/v1/sensors);
- Состояние датчика по ID (через эндпойнт GET /api/v1/sensors/:id); 
- Состояние датчика температуры по его местоположению (через эндпойнт GET /api/v1/sensors/temperature/:location).  
Система поддерживает только датчики температуры
```
// SensorType represents the type of sensor
type SensorType string
const (
	Temperature SensorType = "temperature"
)
```

### 2. Анализ архитектуры монолитного приложения

Язык программирования: Go.  
База данных: PostgreSQL.  
Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.  
Взаимодействие: Синхронное, запросы обрабатываются последовательно.  
Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.  
Развертывание: Требует остановки всего приложения.  

### 3. Определение доменов и границы контекстов

В рамках методологии DDD у существующего приложения можно выделить домен:  
- Домен датчиков (Sensors Domain) – контроль статуса и показаний датчиков; 
Для полноценного приложения его необходимо расширить перечень доменов, добавив:  
- Домен пользователей (Users Domain) – аутентификация пользователей, авторизация доступа и действий, производимых пользователями, хранение пользовательских настроек, средства информирования пользователей; 
- Домен платежей (Payments Domain) – выставление счетов, контроль состояния баланса, блокировка использования сервиса при “отрицательном балансе”, пользовательский интерфейс для оплаты, взаимодействие с платежными системами; 
- Домен автоматизации (Automation Domain) – обеспечение выполнения периодических действий (по таймеру), простых сценариев (если, то) и сложных сценариев управления автоматикой умного дома.   
Домен датчиков (Sensors Domain) должен быть расширен до Домена устройств (Devices Domain) – контроль статуса датчиков и исполнительных устройств, получение показаний датчиков, хранение трендов (временных рядов показаний), управление исполнительными устройствами, аварийные блокировки, обновление микрокодов и прошивок.  

### **4. Проблемы монолитного решения**

У текущего решения есть ряд проблем  
- Недостаточный функционал для управления умным домом (необходимы, как минимум: расширение типов устройств, возможность управления исполнительными устройствами, автоматизация управления);
- Отсутствие средств обеспечения безопасности (как минимум: аутентификация и авторизация, проверка вводимых уставок на допустимость значений, контроль допустимых режимов работы); 
- Развитие продукта потребует совместной работы нескольких команд разработчиков по различным направлениям (управление пользователями, платежи, автоматизация, расширение управления устройствами), разумно предложить переход от монолитного приложения к микросервисам; 
- Использование одной БД для хранения разнородных данных (настройки системы, аутентификационная информация, платежная информация, временные ряды) с различными требованиями по безопасности и производительности неэффективно и небезопасно; 
- Существующая схема неустойчиво к отказам, в случае возникновения проблемы с любой частью системы будет потеряна вся функциональность для всех пользователей.

### 5. Визуализация контекста системы — диаграмма С4

```markdown
[Диаграмма контекста в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/context/Warmhouse_Context.puml)
```

# Задание 2. Проектирование микросервисной архитектуры

В этом задании вам нужно предоставить только диаграммы в модели C4. Мы не просим вас отдельно описывать получившиеся микросервисы и то, как вы определили взаимодействия между компонентами To-Be системы. Если вы правильно подготовите диаграммы C4, они и так это покажут.

**Диаграмма контейнеров (Containers)**

```markdown
[Диаграмма контейнеров в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/containers/Warmhouse_Container.puml)
```

**Диаграмма компонентов (Components)**

```markdown
[Диаграмма компонента Auth в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Auth.puml)
[Диаграмма компонента Automation в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Automation.puml)
[Диаграмма компонента Device в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Device.puml)
[Диаграмма компонента Logging в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Logging.puml)
[Диаграмма компонента Notification в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Notification.puml)
[Диаграмма компонента Payment в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_Payment.puml)
[Диаграмма компонента User в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/components/Warmhouse_Component_User.puml)
```

**Диаграмма кода (Code)**

```markdown
[Диаграмма кода контейнера UserService в модели C4](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/c4/code/Warmhouse_Code_User.puml)
```

# Задание 3. Разработка ER-диаграммы

```markdown
[ER-диаграмма](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/er/ER_diagram.puml)
```

# Задание 4. Создание и документирование API

### 1. Тип API

Будет использован гибридный подход. Взаимодействие между микросервисами и с пользователями, требующее быстрого отклика или строго консистентного состояния (доступ пользователя после проверки прав), будет производиться с использованием синхронное взаимодействие (REST/HTTPS). Передача сообщений (логирование, нотификация) и событий (показания датчиков, управляющие команды) будут производиться асинхронно (Message Queue).

### 2. Документация API

```markdown
[API микросервиса Auth](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/api/Warmhouse_Auth_Service.yaml)
[API микросервиса Automation](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/api/Warmhouse_Automation_Service.yaml)
[API микросервиса Device](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/api/Warmhouse_Device_Service.yaml)
[API микросервиса User](https://github.com/alexeit-315/warmhouse/blob/main/docs/diagrams/api/Warmhouse_User_Service.yaml)
```

# Задание 5. Работа с docker и docker-compose

Перейдите в apps.

Там находится приложение-монолит для работы с датчиками температуры. В README.md описано как запустить решение.

Вам нужно:

1) сделать простое приложение temperature-api на любом удобном для вас языке программирования, которое при запросе /temperature?location= будет отдавать рандомное значение температуры.

Locations - название комнаты, sensorId - идентификатор названия комнаты

```
	// If no location is provided, use a default based on sensor ID
	if location == "" {
		switch sensorID {
		case "1":
			location = "Living Room"
		case "2":
			location = "Bedroom"
		case "3":
			location = "Kitchen"
		default:
			location = "Unknown"
		}
	}

	// If no sensor ID is provided, generate one based on location
	if sensorID == "" {
		switch location {
		case "Living Room":
			sensorID = "1"
		case "Bedroom":
			sensorID = "2"
		case "Kitchen":
			sensorID = "3"
		default:
			sensorID = "0"
		}
	}
```

2) Приложение следует упаковать в Docker и добавить в docker-compose. Порт по умолчанию должен быть 8081

3) Кроме того для smart_home приложения требуется база данных - добавьте в docker-compose файл настройки для запуска postgres с указанием скрипта инициализации ./smart_home/init.sql

Для проверки можно использовать Postman коллекцию smarthome-api.postman_collection.json и вызвать:

- Create Sensor
- Get All Sensors

Должно при каждом вызове отображаться разное значение температуры

Ревьюер будет проверять точно так же.


