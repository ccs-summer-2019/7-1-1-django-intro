## STEP ONE

Create a project folder, move into the project folder, and install Django.
```
mkdir <project_name>
cd <project_name>
pipenv install django
```
## STEP TWO

Activate the shell
```
pipenv shell
```

## STEP THREE

Start a new Django project ¡DON'T FORGET THE PERIOD!
```
django-admin startproject <project_name> .
```
Let's take a look at what `startproject` created for you.

## STEP FOUR

Verify your Django project works.
```
python manage.py runserver
```

Django created migrations when you setup your project. It's a good idea at this point to go ahead apply those migrations. Run the following command in the terminal:
```
python manage.py migrate
```

The migrate command looks at the **INSTALLED_APPS**  setting and creates any necessary database tables.

## STEP FIVE

Create an application.
```
python manage.py startapp <app_name>
```
Tell Django to use the app you created by adding it to the list of **INSTALLED APPS** in the `<project_name>/settings.py` file:
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    '<app_name>', // newly create app name goes here
]
```
## STEP SIX

Let’s write the first view. Open the file `polls/views.py` and put the following Python code in it:
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
This is the simplest view possible in Django. To call the view, we need to map it to a URL - and for this we need a URLconf.

## STEP SEVEN

To create a URLconf in the polls directory, create a file called urls.py.

In the polls/urls.py file include the following code:
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
## STEP EIGHT

Point the root URLconf at the `polls.urls` module. In `mysite/urls.py`, add an import for `django.urls.include` and insert an `include()` in the `urlpatterns` list:
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
## STEP NINE

Define your models in the `<app_name>/models.py`:
```
from django.db import models
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    # you can use an optional first positional arg to a Field
    # to designate a human-readable format
    pub_date = models.DateTimeField('date published', default=timezone.now)


    def __str__(self):
          return self.question_text

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
          return self.choice_text
```
Now we need to **makemigrations**.
```
python manage.py makemigrations polls
```
Migrations are how Django stores changes to your models.

Now, let's run the migration to our database.
```
python manage.py migrate
```
Essentially, this syncs the changes you've made to your models with the schema in the database.

## STEP TEN

Register the models. Add the following to `polls/admin.py`
```
from django.contrib import admin

from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```
Let's check out the Django admin:
```
python manage.py createsuperuser
```
Start the server and navigate to `http://127.0.0.8000/admin`.

## LET'S WRITE SOME MORE VIEWS

```
def index(request):
    return HttpResponse("Hello, world!")

def detail(request, question_id):
    return HttpResponse("Details for question #%s" % question_id)

def results(request, question_id):
    return HttpResponse("Results for question #%s" % question_id)

def vote(request, question_id):
    return HttpResponse("Voting for question #%s" % question_id)
```

## WIRE THE VIEWS INTO THE PROJECT

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
    # variable name url pattern
    # will be sent as a keyword arg to the view
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),

]
```

## WRITE VIEWS THAT PULL DATA FROM THE DATABASE

## TIME FOR TEMPLATES

Create a templates/polls directory in your polls app - by convention, DjangoTemplates looks for a templates subdirectory in each of the **INSTALLED_APPS**.
```
{% if question_list %}

  <ul>
    {% for question in question_list %}
    <li><a href="/polls/{{ question.id }}/">{{question.question_text}}</a></li>
    {% endfor %}
  </ul>

{% else %}

  <p>No polls available</p>

{% endif %}
```
## LET'S UPDATE OUR VIEWS
```
from django.shortcuts import render

def index(request):
    question_list = Question.objects.all()
    context = {
        'question_list': question_list
    }
    return render(request, 'polls/index.html', context)
```
## CREATE A DETAIL VIEW
```
from django.shortcuts import render, get_object_or_404

def detail(request, question_id):
    # question = Question.objects.get(pk=question_id)
    question = get_object_or_404(Question, pk=question_id)
    context = {'question': question}
    return render(request, 'polls/detail.html', context)
```
## ADD STYLES

Create a static/polls directory in your polls app and add a `styles.css` file.
```
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/styles.css' %}">
```
