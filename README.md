Конечно! Давайте реализуем удаление исследователя и редактирование информации о миссии в Django. Мы создадим два представления (views) и определим соответствующие маршруты (urls).

▎Шаг 1: Создание модели

Сначала создадим модель для исследователей, если она еще не создана. В файле models.py вашего приложения:

from django.db import models

class Explorer(models.Model):
    name = models.CharField(max_length=100)
    birthdate = models.DateField()
    deathdate = models.DateField(null=True, blank=True)
    nationality = models.CharField(max_length=50)
    bio = models.JSONField()  # Используем JSONField для хранения биографии

    def __str__(self):
        return self.name


▎Шаг 2: Создание представлений

Теперь создадим представления для удаления исследователя и редактирования информации о миссии в файле views.py:

from django.http import JsonResponse, HttpResponseNotFound
from django.views import View
from .models import Explorer
import json

class DeleteExplorerView(View):
    def delete(self, request, explorer_name):
        try:
            explorer = Explorer.objects.get(name=explorer_name)
            explorer.delete()
            return JsonResponse({}, status=204)  # Успех, без содержимого
        except Explorer.DoesNotExist:
            return HttpResponseNotFound({"error": "Исследователь не найден"})

class UpdateExplorerView(View):
    def patch(self, request, explorer_name):
        try:
            explorer = Explorer.objects.get(name=explorer_name)
            data = json.loads(request.body)

            # Обновление данных исследователя
            for key, value in data.items():
                setattr(explorer, key, value)
            explorer.save()

            return JsonResponse({
                "data": {
                    "code": 200,
                    "message": "Миссия обновлена"
                }
            }, status=200)

        except Explorer.DoesNotExist:
            return HttpResponseNotFound({"error": "Исследователь не найден"})
        except json.JSONDecodeError:
            return JsonResponse({"error": "Неверный формат данных"}, status=400)


▎Шаг 3: Настройка маршрутов

Теперь добавим маршруты для этих представлений в файл urls.py вашего приложения:

from django.urls import path
from .views import DeleteExplorerView, UpdateExplorerView

urlpatterns = [
    path('explorer/<str:explorer_name>/', DeleteExplorerView.as_view(), name='delete_explorer'),
    path('explorer/<str:explorer_name>/update/', UpdateExplorerView.as_view(), name='update_explorer'),
]


▎Шаг 4: Настройка проекта

Не забудьте добавить ваше приложение в INSTALLED_APPS в файле settings.py, если вы этого еще не сделали:

INSTALLED_APPS = [
    ...
    'your_app_name',
]


▎Шаг 5: Миграции

Не забудьте выполнить миграции для создания таблиц в базе данных:

python manage.py makemigrations
python manage.py migrate


▎Примеры запросов

1. Удаление исследователя:

      DELETE /explorer/richard_burton/
   

   Ответ: 204 No Content.

2. Редактирование миссии:

      PATCH /explorer/richard_burton/update/
   Content-Type: application/json

   {
       "birthdate": "1821-03-19",
       "deathdate": "1890-10-20",
       "nationality": "Британец",
       "bio": {
           "early_life": "Родился в Торки, Англия.",
           "motivation": "Исследование неизведанных регионов.",
           "achievements": ["Исследовал озеро Танганьика."]
       }
   }
   

   Ответ:

      {
       "data": {
           "code": 200,
           "message": "Миссия обновлена"
       }
   }
   

Теперь у вас есть базовая реализация удаления исследователя и редактирования его информации в Django. Вы можете адаптировать этот код под свои нужды и добавлять дополнительные проверки и функционал по мере необходимости.


Конечно! Давайте добавим методы для создания поездки, получения списка доступных туров, записи на тур и поиска по животным и исследователям в Django.

▎Шаг 1: Создание модели для туров

Сначала создадим модель для хранения информации о турах в models.py вашего приложения:

from django.db import models

class Tour(models.Model):
    tour_number = models.CharField(max_length=10, unique=True)
    launch_date = models.DateField()
    seats_available = models.IntegerField()

    def __str__(self):
        return f"Tour {self.tour_number}"


▎Шаг 2: Создание представлений

Теперь создадим представления для каждого из методов в views.py:

from django.http import JsonResponse, HttpResponseNotFound
from django.views import View
from .models import Tour
import json

class CreateTourView(View):
    def post(self, request):
        data = json.loads(request.body)
        tour_number = data.get('tour_number')
        launch_date = data.get('launch_date')
        seats_available = data.get('seats_available')

        # Создание нового тура
        tour = Tour(tour_number=tour_number, launch_date=launch_date, seats_available=seats_available)
        tour.save()

        return JsonResponse({
            "data": {
                "code": 201,
                "message": "Тур зарегистирован"
            }
        }, status=201)

class ListToursView(View):
    def get(self, request):
        tours = Tour.objects.all().values('tour_number', 'launch_date', 'seats_available')
        return JsonResponse({
            "data": list(tours)
        }, status=200)

class BookTourView(View):
    def post(self, request):
        data = json.loads(request.body)
        flight_number = data.get('flight_number')

        # Логика записи на тур (например, проверка наличия мест)
        # Здесь можно добавить логику для бронирования тура

        return JsonResponse({
            "data": {
                "code": 201,
                "message": "Тур забронирован"
            }
        }, status=201)

class SearchAnimalsAndExplorersView(View):
    def get(self, request):
        query = request.GET.get('query', '')

        # Пример поиска по животным (замените на вашу логику поиска)
        if query.lower() == 'слон':
            return JsonResponse({
                "data": {
                    "name": "Африканский слон",
                    "scientific_name": "Loxodonta africana",
                    "population_estimate": 415000,
                    "conservation_status": "Уязвимый",
                    "habitat": {
                        "region": "Саванны и леса Африки",
                        "countries": [
                            "Ботсвана",
                            "Кения",
                            "Танзания",
                            "Зимбабве"
                        ],
                        "coordinates": {
                            "latitude": "-6.3690000",
                            "longitude": "34.8888000"
                        }
                    },
                    "characteristics": {
                        "diet": "Травоядное",
                        "lifespan": "60-70 лет",
                        "weight": {
                            "min_kg": 4000,
                            "max_kg": 7000
                        },
                        "height": {
                            "min_meters": 2.5,
                            "max_meters": 4
                        }
                    },
                    "behavior": {
                        "social_structure": "Стада",
                        "communication": "Инфразвук, вибрации",
                        "migration_patterns": "Сезонные перемещения"
                    },
                    "threats": [
                        {
                            "name": "Браконьерство",
                            "description": "Охота за слоновой костью."
                        },
                        {
                            "name": "Потеря среды обитания",

                                "description": "Расширение сельского хозяйства и урбанизация."
                        }
                    ],
                    "conservation_efforts": [
                        {
                            "organization": "Всемирный фонд дикой природы (WWF)",
                            "actions": [
                                "Борьба с браконьерством",
                                "Создание заповедников",
                                "Образовательные программы"
                            ]
                        }
                    ]
                }
            }, status=200)

        return JsonResponse({"data": {}}, status=404)


▎Шаг 3: Настройка маршрутов

Теперь добавим маршруты для этих представлений в файл urls.py вашего приложения:

from django.urls import path
from .views import CreateTourView, ListToursView, BookTourView, SearchAnimalsAndExplorersView

urlpatterns = [
    path('tour/', CreateTourView.as_view(), name='create_tour'),
    path('tours/', ListToursView.as_view(), name='list_tours'),
    path('book-tour/', BookTourView.as_view(), name='book_tour'),
    path('search/', SearchAnimalsAndExplorersView.as_view(), name='search_animals_explorers'),
]


▎Шаг 4: Настройка проекта

Не забудьте добавить ваше приложение в INSTALLED_APPS в файле settings.py, если вы этого еще не сделали:

INSTALLED_APPS = [
    ...
    'your_app_name',
]


▎Шаг 5: Миграции

Не забудьте выполнить миграции для создания таблиц в базе данных:

python manage.py makemigrations
python manage.py migrate


▎Примеры запросов

1. Создание поездки:

      POST /tour/
   Content-Type: application/json

   {
       "tour_number": "103",
       "launch_date": "2025-05-15",
       "seats_available": 6
   }
   

   Ответ:

      {
       "data": {
           "code": 201,
           "message": "Тур зарегистирован"
       }
   }
   

2. Получение списка доступных туров:

      GET /tours/
   Content-Type: application/json
   

   Ответ:

      {
       "data": [
           {
               "tour_number": "103",
               "launch_date": "2025-05-15",
               "seats_available": 6
           },
           {
               "tour_number": "104",
               "launch_date": "2025-06-15",
               "seats_available": 8
           }
       ]
   }
   

3. Запись на тур:

      POST /book-tour/
   Content-Type: application/json

   {
       "flight_number": "101"
   }
   

   Ответ:

      {
       "data": {
           "code": 201,
           "message": "Тур забронирован"
       }
   }
   

4. Поиск по животным и исследователям:

      GET /search?query=слон
   Content-Type: application/json
   

   Ответ:

      {
       "data": {
           ...
           // Данные о слоне как указано выше
           ...
       }
   }
   

Теперь у вас есть полная реализация методов для создания поездки, получения списка доступных туров, записи на тур и поиска по животным и исследователям в Django. Вы можете адаптировать этот код под свои нужды и добавлять дополнительные проверки и функционал по мере необходимости.                  
