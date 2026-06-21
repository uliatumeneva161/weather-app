# 🌤 Weather Forecast App
<div align="center">
  <video controls width="800" autoplay muted loop poster="src/assets/preview/preview-poster.jpg">
    <source src="/preview/preview.mp4" type="video/mp4">
    Ваш браузер не поддерживает воспроизведение видео.
  </video>
</div>

## 📌 О проекте
Vue.js приложение для просмотра текущей погоды. Получает данные от [WeatherAPI](https://www.weatherapi.com/) с частотой обновления 15 минут.

## ✨ Ключевые особенности
- Прогноз в реальном времени с учетом часового пояса
- Подсказки локации при вводе в строку поиска
- Адаптивный дизайн для мобильных устройств и десктопов

## 🛠 Технологии
| Категория       | Технологии                          |
|-----------------|-------------------------------------|
| Frontend        | Vue 3 (Composition API), Pinia      |
| Стили           | CSS-анимации                        |
| API             | WeatherAPI, Geolocation API         |
| Инструменты     | Vite, ESLint, Prettier              |

## 🚀 Запуск проекта

### 1. Требования
- Node.js v16+
- API ключ от [WeatherAPI](https://www.weatherapi.com/) (необходимо зарегистрироваться и получить ключ)

### 2. Установка зависимостей
Склонируйте репозиторий и установите зависимости:
```bash
git clone https://github.com/ваш-репозиторий.git
cd weather-app
npm install
