# Scalable Reactive Messenger ⚡
**High-load мессенджер на реактивных микросервисах · Java 21 · Spring WebFlux · Kafka · Kubernetes**

[![Java 21](https://img.shields.io/badge/Java-21-red?logo=openjdk)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3-brightgreen?logo=springboot)](https://spring.io/projects/spring-boot)
[![WebFlux](https://img.shields.io/badge/WebFlux-Reactive-blue)](https://spring.io/reactive)
[![Kafka](https://img.shields.io/badge/Kafka-orange?logo=apachekafka)](https://kafka.apache.org/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-blue?logo=kubernetes)](https://kubernetes.io/)
[![Redis](https://img.shields.io/badge/Redis-red?logo=redis)](https://redis.io/)
[![Cassandra](https://img.shields.io/badge/Cassandra-purple?logo=apachecassandra)](https://cassandra.apache.org/)

**Прошлый репозиторий:** https://github.com/MorozovDima07/messenger

### Обзор проекта
Это масштабируемый мессенджер, ориентированный на обработку миллионов пользователей с минимальными задержками. Система обеспечивает real-time коммуникацию, управление профилями, чатами, медиафайлами и уведомлениями.

**Цель:** Максимальная масштабируемость и распределение нагрузки для high-load сценариев (например, 100k+ одновременных WebSocket-соединений, доставка сообщений <100 мс). Проект — мой pet-project, чтобы на практике разобраться, как устроены Telegram/WhatsApp, и набить руку на distributed systems. Я начал с базового мессенджера, но перестроил под реактивную архитектуру с микросервисами для практики в production-grade технологиях.

Проект разбит на несколько репозиториев (микросервисы), но этот — витрина с общим описанием, диаграммами и ссылками. Для удобства вся основная информация собрана в этом README.

### Технологический стек
- **Язык:** Java 21
- **Фреймворки:** Spring WebFlux, Spring Boot, Reactive Spring Security (JWT)
- **Базы данных:** PostgreSQL (R2DBC + Flyway), Apache Cassandra (Reactive, с bucketing/clustering keys), Redis (Reactive для кэша и Pub/Sub), Elasticsearch (для поиска, под вопросом альтернативы в РФ)
- **Очереди/Брокеры:** Apache Kafka (Reactor Kafka для событий), Redis Pub/Sub (для быстрой доставки)
- **Безопасность:** Bouncy Castle (AES-256), JWT с ротацией ключей (планирую HashiCorp Vault)
- **Облако:** AWS S3/CloudFront (для медиа, альтернативы под вопросом), Firebase Cloud Messaging (FCM для пушей)
- **Мониторинг:** Micrometer, Prometheus, Grafana, ELK Stack, Jaeger (для трассировки, под вопросом)
- **API Gateway:** Envoy (для sticky sessions и маршрутизации)
- **Инфраструктура:** Docker, Kubernetes (с операторами для кластеров), gRPC (для асинхронной коммуникации между сервисами)
- **CI/CD:** GitHub Actions
- **Тестирование:** JUnit, Testcontainers
- **Управление версиями:** Git
- **Frontend:** React Native (в разработке для мобильного клиента)

Дополнительно: Retry, Circuit Breaker, fallback для отказоустойчивости; MessageEnvelope как универсальный DTO для событий (сообщения, ACK, read/delivered, typing, online).

### Архитектура
Система — микросервисная, реактивная, event-driven.

**Диаграмма отправки сообщения (svg и png):** ![Диаграмма отправки сообщения в svg](Диаграмма%20последовательности%20для%20отправки%20сообщения.svg)
![Диаграмма отправки сообщения в png](Диаграмма%20последовательности%20для%20отправки%20сообщения.png)

**Общая архитектура:** ![Архитектура мессенджера](Архитектура%20мессенджера.png)

#### Микросервисы
1. **Auth Service** — в разработке  
   Регистрация, логин, JWT ротация с Redis locks. (Учебный, переделаю с Vault).
2. **User Service** — в разработке  
   Профили, контакты, поиск (PostgreSQL с шардированием в планах).
3. **Edge Service** — [https://github.com/Dmitry-Morozov-Dev/edge-service](https://github.com/Dmitry-Morozov-Dev/edge-service)  
   WebSocket, heartbeat, сессии в Caffeine cache. Sticky sessions via Envoy.
5. **Delivery Worker** — [https://github.com/Dmitry-Morozov-Dev/delivery-worker-service ](https://github.com/Dmitry-Morozov-Dev/delivery-worker-service)  
   Фан-аут, маршрутизация по Redis mappings. Graceful shutdown via Kafka.
7. **Chat Service** — https://github.com/Dmitry-Morozov-Dev/chat-service  
   Хранение в Cassandra (9 таблиц с bucketing), поиск в Elasticsearch, кэш в Redis.
9. **Notification Service** — в планах  
   Пушы для оффлайн (FCM, токены в Cassandra).
10. **Media Service** — в разработке  
   Загрузка/URL в S3, thumbnails.
11. **Analytics/Monitoring** — в планах  
   Метрики, логи.

### Что уже реализовано
- Edge, Delivery Worker, Chat Service: WebSocket-соединения, события через Kafka/Redis Pub/Sub, сохранение в Cassandra/Redis.
- Отправка сообщений, статусы (delivered/read), typing, online.
- Создание/получение чатов via HTTP.
- Docker-compose для локального запуска (Kafka, Redis, Cassandra, PostgreSQL, Envoy).
- Базовые метрики; retry/Circuit Breaker.
- Frontend на React Native: Основной функционал (чаты, сообщения).

**Демо:** [Смотреть демо на YouTube](https://youtu.be/asuP2z5bgkI) 
[![Смотреть демо на YouTube — кликни на превью](https://img.youtube.com/vi/asuP2z5bgkI/maxresdefault.jpg)](https://youtu.be/asuP2z5bgkI)  
Пользовательская часть (авторизация) в разработке.

### Запуск локально
1. Клонируйте репо  
   `git clone https://github.com/Dmitry-Morozov-Dev/chat-service.git
   cd chat-service`
2. Запустите  
   `docker-compose up --build`
3. Подключитесь  
   WebSocket: ws://localhost:8080/ws/connect  
   API: http://localhost:8082

**Деплой:** Kubernetes с операторами (протестировано); CI/CD via GitHub Actions (базовый).

### Схемы данных (Cassandra в Chat Service)
- BlockedUserByChat / Chat / Participant / Media / Message / UserChatsInfo / List — с bucketing (bucketMonth/Partition) для масштаба.
- Elasticsearch: Для поиска сообщений/пользователей (отдельные индексы по языкам в планах).
- PostgreSQL: User/Device в Auth/User Services.

### Сценарии (Use Cases)
- **Отправка сообщения:**  
  Клиент → Edge (WebSocket) → Kafka → Delivery Worker (Redis mappings) → Pub/Sub → Edge (доставка) + Chat Service (сохранение) + Notifications (пуш оффлайн).

### Планы и примечания
- Добавить Elasticsearch, Jaeger, полный мониторинг.
- Оптимизации: Локальный кэш в Delivery Worker; CRDT в Cassandra.
- Фронтенд: Полный мобильный клиент.
- Git: Коммиты не всегда структурированные (из-за изучения технологий), но улучшаю.

Это мой pet-project для поиска первой работы Java Backend. Готов к собеседованиям!

Если проект полезен — ⭐ star! Помогает при job hunt.

✉️ dmitry.morozov.dev@gmail.com | 🌍 t.me/dmitry_morozov_dev | GitHub: Dmitry-Morozov-Dev

# Dmitry Morozov 👋
**Junior Java Backend Developer** · Ищу первую работу · Строю high-load мессенджер

🔥 Главный проект: Масштабируемый реактивный чат-сервис на микросервисах  
→ https://github.com/Dmitry-Morozov-Dev/scalable-reactive-messenger ← (с видео демо и диаграммами)

### Ищу
Первую позицию Java Backend. Готов учиться, писать код и масштабировать системы.

📫 dmitry.morozov.dev@outlook.com
✈️ [t.me/dmitry_morozov_dev](https://t.me/DmitryMorozov07)

Готов к удалёнке. Свяжитесь для деталей!
